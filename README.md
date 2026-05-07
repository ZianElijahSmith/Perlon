# Perlon
An attempt to make a Perl Interpreter similar to Ipython, in Perl, for Perl.
<br>
Some of this code made with AI, some of it is not.
<br><br>
<img width="559" height="294" alt="image" src="https://github.com/user-attachments/assets/3e331c41-9aa0-45a7-af00-f4639f01843b" />


# Current Features
0. Beloved color coded syntax for easier readability.
1. Can use up and/or down arrows to go through code history **(useful for fixing typos without retyping your entire line)**
   
   **History is saved to ~/.perlon_history and reloaded on next launch, so up-arrow works across sessions**
   
   Consecutive duplicate lines aren't saved to history (so spamming the same command doesn't pollute it)
3. Ctrl-C exits cleanly (saves history first)
4. Finally, multi-line support. If you open a {, it'll "..." until you close.


# Features that will probably be implemented
0. being able to type in terminal commands within the perlon shell via ! (example: !ls, !ping)
