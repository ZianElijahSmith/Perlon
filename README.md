# IPerl REPL

An attempt to make a Perl REPL similar to Ipython, in Perl, for Perl. (We call it IPerl).<br>
(Was originally named Perlon, was renamed after "Perlon AI" became a big thing)
<br><br>
Some of this code was made with AI, some of it was not.<br>
Contributions and suggestions welcomed.
<br><br>
<img width="566" height="299" alt="image" src="https://github.com/user-attachments/assets/f2deb4e1-1a23-4b55-af1a-49e71ef73c39" />



# Current Features
0. Beloved color coded syntax for easier readability.
1. Can use up and/or down arrows to go through code history **(useful for fixing typos without retyping your entire line)**
   
   **History is saved to ~/.perlon_history and reloaded on next launch, so up-arrow works across sessions**
   
   Consecutive duplicate lines aren't saved to history (so spamming the same command doesn't pollute it)

   left and/or right arrows allow you to traverse code as well (obviously)
   
3. Ctrl-C exits cleanly (saves history first)
4. We finally have multi-line support. If you open a {, it'll "..." until you close with }.


# Features that will probably be implemented
0. being able to type in terminal commands within the perlon shell via ! (example: !ls, !ping)

# Disclaimer
I openly admit Perl is not my strongest language, but it is a language I love.
I think Perl would be more popular if both oldschool users and new learners had a fun REPL to use.
Using Ipython **really** helped me improve my understanding of python and was enjoyable.
If you have the skills and knowledge to do IPerl justice, feel free to make a suggestion.
Until then I will update this in small bites from time to time.
