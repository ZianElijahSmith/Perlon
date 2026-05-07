# Perlon
An attempt to make a Perl Interpreter similar to Ipython, for Perl, and in Perl.
<br>
Some of this code made with AI, some of it is not.
<br><br>
<img width="563" height="147" alt="image" src="https://github.com/user-attachments/assets/c964f80d-31ba-49cc-ae08-318d860564f0" />

# Current Features
0. Beloved color coded syntax for easier readability.
1. Can use up and/or down arrows to go through code history **(useful for fixing typos without retyping your entire line)**
   
   History is saved to ~/.perlon_history and reloaded on next launch, so up-arrow works across sessions
   Consecutive duplicate lines aren't saved to history (so spamming the same command doesn't pollute it)
3. Ctrl-C exits cleanly (saves history first)
4. Finally, multi-line support. If you open a {, it'll "..." until you close.


# Features that will probably be implemented
0. being able to type in terminal commands within the perlon shell via ! (example: !ls, !ping)
