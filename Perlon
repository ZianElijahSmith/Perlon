#!/usr/bin/env perl

# Perl is kinda weird for me, and implementing certain features has be done totally different ways
# For example, 'use strict' will affect how perlon works, but "use strict" doesn't exist in python

use strict;
use warnings;
use Term::ReadKey;
use Term::ANSIColor qw(:constants);
use Data::Dumper;
use feature 'say';

# ── History ────────────────────────────────────────────────────────────────────
my $history_file = "$ENV{HOME}/.perlon_history";
my @history;
if (-f $history_file) {
    open my $fh, '<', $history_file or warn "Can't read history: $!\n";
    @history = <$fh>;
    chomp @history;
    close $fh;
}

sub save_history {
    open my $fh, '>', $history_file or warn "Couldn't save history: $!\n";
    say $fh $_ for @history;
    close $fh;
}

# ── Syntax Highlighting ────────────────────────────────────────────────────────
my %color_rules = (
    # Built-ins / keywords
    '\b(sub|use|package|if|else|elsif|for|foreach|while|until|do|'.
    'say|print|printf|die|warn|eval|return|last|next|redo|'.
    'push|pop|shift|unshift|splice|keys|values|each|'.
    'exists|delete|defined|undef|ref|bless|tied|'.
    'chomp|chop|join|split|map|grep|sort|reverse|'.
    'open|close|read|write|seek|tell|eof|'.
    'length|substr|index|rindex|sprintf|'.
    'scalar|wantarray|caller|local|our|state)\b'
                                            => BOLD . CYAN,
    '\b(my)\b'                              => BOLD . GREEN,

    # Sigils  (order: scalars before @ so $a in @array doesn't double-match)
    '(\$[a-zA-Z_]\w*)'                      => BOLD . RED,
    '(\@[a-zA-Z_]\w*)'                      => BOLD . BLUE,
    '(\%[a-zA-Z_]\w*)'                      => BOLD . MAGENTA,

    # Literals
    '(\b\d+(?:\.\d+)?\b)'                   => BOLD . YELLOW,
    '("(?:[^"\\\\]|\\\\.)*"|\'(?:[^\'\\\\]|\\\\.)*\')' => BOLD . YELLOW,

    # Operators & punctuation
    '(=>|->|=~|!~|\.\.\.?|&&|\|\||!!|[+\-*\/%=<>!&|^~.]+)' => YELLOW,
    '([\(\)\{\}\[\];,])'                    => WHITE,
);

sub highlight {
    my ($text) = @_;
    return "" unless defined $text;

    # --- Single-pass approach ---
    # Collect all matches against the ORIGINAL string first, so that ANSI
    # codes added for one match can never be re-matched by a later pattern.
    my @spans;   # ( { start, end, color } )

    for my $pat (sort { length($b) <=> length($a) } keys %color_rules) {
        my $col = $color_rules{$pat};
        while ($text =~ /$pat/g) {
            # $-[1]/$+[1] are start/end of capture group 1
            my ($s, $e) = ($-[1], $+[1]);
            next unless defined $s;

            # Skip if this region already claimed by an earlier (longer) pattern
            my $overlap = 0;
            for my $sp (@spans) {
                if ($s < $sp->{end} && $e > $sp->{start}) {
                    $overlap = 1; last;
                }
            }
            push @spans, { start => $s, end => $e, color => $col }
                unless $overlap;
        }
    }

    # Sort by position in the string
    @spans = sort { $a->{start} <=> $b->{start} } @spans;

    # Assemble: plain text + color-wrapped spans, one pass
    my $out = "";
    my $pos = 0;
    for my $sp (@spans) {
        $out .= substr($text, $pos, $sp->{start} - $pos);   # gap before match
        $out .= $sp->{color}
              . substr($text, $sp->{start}, $sp->{end} - $sp->{start})
              . RESET;
        $pos = $sp->{end};
    }
    $out .= substr($text, $pos);   # tail after last match

    return $out;
}

# ── Input Engine ───────────────────────────────────────────────────────────────
# Reads a full line with:
#   • Live syntax highlighting
#   • Left / Right cursor movement
#   • Up / Down history browsing
#   • Home (Ctrl-A) / End (Ctrl-E)
#   • Ctrl-C to exit cleanly
sub get_live_line {
    my ($prompt) = @_;

    my $input      = "";
    my $cursor     = 0;
    my $hist_idx   = scalar @history;   # one past the end = "current" slot
    my $saved_line = "";                # stash the in-progress line while browsing

    ReadMode('raw');
    print $prompt;

    while (1) {
        my $key = ReadKey(0);
        last unless defined $key;

        my $ord = ord($key);

        # ── Ctrl-C ──────────────────────────────────────────────────────────
        if ($ord == 3) {
            print "\n";
            ReadMode('restore');
            save_history();
            print BOLD, CYAN, "Goodbye!", RESET, "\n";
            exit;
        }

        # ── Enter ───────────────────────────────────────────────────────────
        if ($key eq "\n" || $key eq "\r") {
            print "\n";
            last;
        }

        # ── Escape sequence  (arrows, Delete, etc.) ─────────────────────────
        if ($key eq "\e") {
            # Non-blocking reads: give the terminal ~50 ms to buffer follow-ups
            my $s1 = ReadKey(-1);
            my $s2 = ReadKey(-1);

            if (defined $s1 && $s1 eq '[' && defined $s2) {
                if ($s2 eq 'A') {           # ↑  Up — older history
                    if ($hist_idx > 0) {
                        $saved_line = $input if $hist_idx == scalar @history;
                        $hist_idx--;
                        $input  = $history[$hist_idx];
                        $cursor = length($input);
                    }
                }
                elsif ($s2 eq 'B') {        # ↓  Down — newer history
                    if ($hist_idx < scalar @history) {
                        $hist_idx++;
                        $input  = ($hist_idx == scalar @history)
                                  ? $saved_line
                                  : $history[$hist_idx];
                        $cursor = length($input);
                    }
                }
                elsif ($s2 eq 'C') {        # →  Right
                    $cursor++ if $cursor < length($input);
                }
                elsif ($s2 eq 'D') {        # ←  Left
                    $cursor-- if $cursor > 0;
                }
                elsif ($s2 eq '3') {        # Delete key (ESC [ 3 ~)
                    ReadKey(-1);            # consume the trailing '~'
                    if ($cursor < length($input)) {
                        substr($input, $cursor, 1) = '';
                    }
                }
                elsif ($s2 eq 'H' || $s2 eq '1') {   # Home (some terminals)
                    ReadKey(-1) if $s2 eq '1';
                    $cursor = 0;
                }
                elsif ($s2 eq 'F' || $s2 eq '4') {   # End
                    ReadKey(-1) if $s2 eq '4';
                    $cursor = length($input);
                }
            }
            # Unknown escape — ignore
        }

        # ── Ctrl-A  (Home) ──────────────────────────────────────────────────
        elsif ($ord == 1)  { $cursor = 0; }

        # ── Ctrl-E  (End) ───────────────────────────────────────────────────
        elsif ($ord == 5)  { $cursor = length($input); }

        # ── Ctrl-K  (kill to end) ───────────────────────────────────────────
        elsif ($ord == 11) { $input = substr($input, 0, $cursor); }

        # ── Ctrl-U  (kill to start) ─────────────────────────────────────────
        elsif ($ord == 21) { $input = substr($input, $cursor); $cursor = 0; }

        # ── Backspace ───────────────────────────────────────────────────────
        elsif ($ord == 127 || $ord == 8) {
            if ($cursor > 0) {
                substr($input, $cursor - 1, 1) = '';
                $cursor--;
            }
        }

        # ── Printable character ──────────────────────────────────────────────
        elsif ($ord >= 32) {
            substr($input, $cursor, 0) = $key;
            $cursor++;
        }

        # ── Redraw ──────────────────────────────────────────────────────────
        print "\r\e[K",          # back to column 0, clear line
              $prompt,
              highlight($input);

        # Reposition cursor if it's not at the end of the string
        if ($cursor < length($input)) {
            my $back = length($input) - $cursor;
            print "\e[${back}D";
        }
    }

    ReadMode('restore');
    return $input;
}

# ── Multi-line collector (sub / do blocks) ─────────────────────────────────────
sub collect_block {
    my ($first_line) = @_;
    my $code       = $first_line;
    my $depth      = ($first_line =~ tr/{//) - ($first_line =~ tr/}//);
    while ($depth > 0) {
        my $cont = get_live_line(BOLD . CYAN . "  ... " . RESET);
        $code  .= "\n" . $cont;
        $depth += ($cont =~ tr/{//) - ($cont =~ tr/}//);
    }
    return $code;
}

# ── Main REPL ──────────────────────────────────────────────────────────────────
$Data::Dumper::Indent = 1;
$Data::Dumper::Terse  = 1;
$Data::Dumper::Sortkeys = 1;

print BOLD, CYAN,
      "Perlon - Interactive Perl Shell  (Ctrl-C or 'exit' to quit)",
      RESET, "\n\n";

my $count = 1;

while (1) {
    my $line = get_live_line(BOLD . CYAN . "In [$count]: " . RESET);

    next if $line =~ /^\s*$/;

    if ($line =~ /^\s*exit\s*$/) {
        last;
    }

    # Auto-continue for open braces
    if (($line =~ tr/{//) > ($line =~ tr/}//)) {
        $line = collect_block($line);
    }

    # De-duplicate consecutive identical history entries
    push @history, $line
        unless @history && $history[-1] eq $line;

    # ── Evaluate ─────────────────────────────────────────────────────────────
    my @result;
    my $err;

    {
        local $SIG{__WARN__} = sub {
            print YELLOW, "Warn: ", RESET, @_;
        };
        @result = eval "package main; use feature ':5.10'; no strict 'vars'; no warnings 'redefine'; $line";
        $err    = $@;
    }

    if ($err) {
        # Trim internal eval noise from error message
        (my $clean = $err) =~ s/ at \(eval \d+\) line \d+\.?\s*$//;
        print RED, "Error: ", RESET, $clean, "\n";
    }
    elsif (@result && defined $result[0]) {
        print BOLD, GREEN, "Out [$count]: ", RESET;
        if (ref $result[0]) {
            # Pretty-print references
            print highlight(Dumper(scalar @result == 1 ? $result[0] : \@result)), "\n";
        }
        elsif (scalar @result > 1) {
            # Multiple return values
            say join(", ", @result);
        }
        else {
            say $result[0];
        }
    }

    $count++;
}

save_history();
print BOLD, CYAN, "\nGoodbye!\n", RESET;
