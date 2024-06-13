------------------------------------------------------------------------------------------------------------------------------------------
CS 214 Systems Programming : Project II : My Shell() | Name & NetID : Ayush Munjial [am3002] & Hein Min Thu [hm570]
------------------------------------------------------------------------------------------------------------------------------------------
This project involved designing and implementation of a simple command-line shell, similar to bash or zsh. Our version, mysh, provides 
interactive and batch modes, both of which will read and interpret a sequence of commands. Our implementation includes the following 
files : myshell.c and Makefile. 

This project has utilized tools like posix (unbuffered) stream IO, reading and changing the working directory, spawning child processes 
and obtaining their exit status , use of dup2() and pipe() and reading the contents of a directory.

This shell program is a basic implementation of a command-line interpreter that allows users to execute various commands in a Linux 
environment. It reads input from the user or a batch file, tokenizes it, and then executes the commands accordingly. The shell program 
also supports simple I/O redirection and pipeline operations.

The program consists of the main function and several helper functions that perform various tasks, such as parsing the input, executing 
commands, and storing arguments. The main function runs an infinite loop that reads input from the user or a batch file, tokenizes it, 
and then executes the corresponding commands. If an error occurs during the execution of a command, the program will print an error 
message and continue to the next command.

Moreover, this project implements two (2) of the extensions described in section 3 of the write-up. The extentions we included are : 
------------------------------------------------------------------------------------------------------------------------------------------
Extension 1 : Escape sequences : 
------------------------------------------------------------------------------------------------------------------------------------------
This extension allows for special characters to be escaped in commands by using the backslash '\' character. When a backslash precedes a 
special character, it removes the special handling of that character and allows it to be treated as a regular token character. 

The backslash can be used to create tokens containing spaces, <, >, |, *, and \ characters. Additionally, a backslash followed by a 
newline disables the newline, but does not insert a newline character into the token. Instead, both characters are removed.

Example : foo bar\ baz\<qu\ux\\>spam  // This command contains four tokens: “foo”, “bar baz<quux\”, “>”, and “spam”.
------------------------------------------------------------------------------------------------------------------------------------------
Extension 2 : Home directory : 
------------------------------------------------------------------------------------------------------------------------------------------
This extension uses the environment variable HOME, which contains the user’s home directory, obtained using getenv("HOME"). When the cd 
command is called with no arguments, it changes the working directory to the home directory. Tokens beginning with ~/ are interpreted as 
relative to the home directory, and the ~ is replaced with the home directory.

Example : cd ~/Documents  // This command changes the working directory to the "Documents" folder within the user's home directory.
------------------------------------------------------------------------------------------------------------------------------------------
This README file aims to provide a brief overview of the program and its functionality. The following sections will explain the basic 
usage of the program and provide details on the various features and commands that it supports.
------------------------------------------------------------------------------------------------------------------------------------------
MYSHELL.C : 
------------------------------------------------------------------------------------------------------------------------------------------
int parse_command(char *buffer, char alltokens[NUMOFTOKENS][SIZEOFTOKEN], int *tokenPos, int *charPos);
------------------------------------------------------------------------------------------------------------------------------------------
The parse_command() function takes a character buffer, buffer, as input and extracts tokens from it. Tokens are sequences of characters 
delimited by space, pipe '|', redirect '<' or '>', or wildcard '*' characters.

The function stores the extracted tokens in a 2D character array called alltokens with NUMOFTOKENS rows and SIZEOFTOKEN columns. The 
function also keeps track of the current token being extracted using the tokenPos pointer and the position within the token using the 
charPos pointer.

The function iterates over each character in the buffer and examines it to determine the appropriate action to take. If the character 
is a backslash, it is treated as an escape character, and the next character is added to the current token in the alltokens array. If 
the next character is a newline or the end of the buffer, the function returns -1 to indicate that the input line was not complete.

If the character is an asterisk '*', it is added to the current token as a negative value to indicate a wildcard.

If the character is a space, pipe '|', redirect '<' or '>', the current token is completed and added to the alltokens array. If the 
current token is empty, the function continues to the next character. If the current token is not empty, it is completed by adding a 
null character to the end. The delimiter is also added to the alltokens array as a separate token.

If the character is none of the above, it is a normal character and is added to the current token. Once the function has iterated over 
all characters in the buffer, it checks if the current token is not empty and adds it to the alltokens array with a null character at 
the end. The function then returns the number of tokens extracted, which is stored in NumOfTokens, and resets the tokenPos pointer to 0
for the next use.

Overall, parse_command() is a useful function for parsing command input in a shell program and extracting meaningful tokens for further 
processing.
------------------------------------------------------------------------------------------------------------------------------------------
int execute_Command(int NumOfTokens);
------------------------------------------------------------------------------------------------------------------------------------------
The execute_Command function is a key component of a Unix shell implementation that is responsible for parsing and executing the user's
command line input. The function takes as input the number of tokens in the command line input and iterates over each token in a loop.

During each iteration, the function checks for special characters such as "<", ">", and "|" to handle input/output redirection and piping.
If a pipe character "|" is encountered, the function creates a pipe object to enable inter-process communication between the current and
next sub-command of the pipe.

The subcommand of the pipe is then executed by calling execute_Process function with the appropriate input and output file descriptors.
The exit status of the command is stored in last_command_status variable, which is later used to determine the status of the last
executed command and to print the prompt accordingly.

If a pipe "|" was encountered in the previous iteration, the function sets the read end of the pipe as the input file descriptor for the
next command, and if not, it sets the standard input as the input file descriptor. Similarly, if output redirection ">" was encountered
in the current iteration, the function sets the output file descriptor to the file specified in the command, otherwise it sets the
standard output as the output file descriptor.

At the end of the function, file descriptors are reset to their original values and the last_command_status is returned as the exit status
of the current command. If the user enters the "exit" command, the function exits the shell with a success message.

In summary, the execute_Command function plays a critical role in parsing and executing the user's command line input by handling special
characters and implementing piping and input/output redirection.
------------------------------------------------------------------------------------------------------------------------------------------
int execute_Process(int fd_in, int fd_out, int firstPos, int lastPos);
------------------------------------------------------------------------------------------------------------------------------------------
The function execute_Process takes in four arguments: fd_in, fd_out, firstPos, and lastPos. The function executes a process with the
given arguments.

The function first initializes two variables, last_command_status and argument_Pos. last_command_status is used to store the exit status
of the command executed in the child process, and argument_Pos is used to keep track of the position of the arguments in the arguments 
array.The function then loops through the tokens from firstPos to lastPos, skipping over any pipe, input redirection, output redirection,
or whitespace tokens. It stores each non-skipped token argument in the arguments array using the storeArguments function.

After the argument list has been created, the function checks if the first argument in the array is cd. If it is, the function checks if
there is any argument after cd or not. If there is no argument, the directory is changed to the home directory. Otherwise, the directory 
is changed to the directory specified by the second argument.

If the first argument is not an absolute path, the function searches for the file in several system directories. If the program is found 
in any of these directories, the path to the program is added prior to the first argument in the arguments array.The function then 
creates an array execArguments to hold the arguments that are passed to the execv function. It then forks a child process using fork() 
and sets up file redirection using dup2().

If the pid is 0, then the current process is the child process. The child process executes the command using execv() with the arguments 
in the execArguments array. If the execv() function returns -1, the function prints an error message using perror() and exits with the 
status returned by execv().

If the pid is not 0, then the current process is the parent process. The parent process waits for the child process to complete using 
wait(). The exit status of the current command is returned by the function to print the prompt accordingly.
------------------------------------------------------------------------------------------------------------------------------------------
int storeArguments(char *given_token, char arguments[NUMOFTOKENS][SIZEOFTOKEN], int *argument_Pos);
------------------------------------------------------------------------------------------------------------------------------------------
The storeArguments function is a helper function used in a larger program to process command line arguments. It takes a single token (a 
string) and separates it into its individual components, storing them in an array. If the token contains a wildcard character (represented
by any character with an ASCII value less than zero), the function expands the wildcard and stores each matching filename in the array.

The function takes three arguments: the given token (a string), an array of strings to store the arguments, and a pointer to the current 
position in the argument array. The function returns an integer value indicating the number of matches found for the wildcard, or 1 if 
there was no wildcard. The function starts by iterating backwards through the characters of the given token until it finds a forward slash 
(/) or reaches the beginning of the string. This loop also checks for the presence of any wildcards in the given token. If a wildcard is 
found, a variable containsAsterik is set to 1 to flag the token as containing a wildcard.

If no wildcard is found, the function simply copies the given token to the argument array and increments the argument position pointer. 
Otherwise, the function opens the directory containing the files that match the wildcard, and checks each file in the directory to see 
if it matches the pattern specified by the wildcard.

The function stores the directory path in a variable path_Directory and opens the directory using the opendir function. If the wildcard 
is at the beginning of the token, the path is set to an empty string and the current directory is opened. If the wildcard is not at the 
beginning, the path is set to the part of the token before the wildcard, and the directory at that path is opened.

The function then splits the token into two parts: the part before the wildcard and the part after the wildcard. It stores each part in 
separate arrays, before_Wildcard and after_Wildcard, respectively. These arrays are used to match the filenames in the directory against 
the pattern specified by the wildcard.

The function then reads each file in the directory using the readdir function, and checks each filename to see if it matches the wildcard 
pattern. If the filename matches the pattern, the function adds it to the argument array by concatenating the directory path and filename, 
and storing the resulting string in the argument array.

The function keeps track of the number of matches found using the NumOfMatches variable. If there are no matches, the function simply 
adds the argument to the argument array without expanding the wildcard.

Once all the files in the directory have been checked, the function returns the number of matches found. If no wildcard was present in 
the token, the function returns 1 to indicate that a single argument was added to the argument array. The function also increments the 
argument position pointer after each argument is added to the array, so that subsequent calls to the function will add new arguments to 
the end of the array.
------------------------------------------------------------------------------------------------------------------------------------------
MAKEFILE : This is a simple Makefile with two targets: all and clean.
------------------------------------------------------------------------------------------------------------------------------------------
The all target is used to build the myshell executable. It depends on the myshell.c file, which is the source code for the shell. The 
Makefile specifies that the gcc compiler should be used to compile the myshell.c file with debugging (-g) and warning (-Wall) flags, and
the resulting executable should be named myshell.

The clean target is used to remove all files created during the build process, as well as any generated files such as the build directory.
It specifies that the files myshell, foo, bar, and baz should be deleted, as well as the build directory.

To execute it the following steps are performed in the Terminal as shown below:

am3002@kill:~/Downloads/214 Systems Programming/pa2$ make
am3002@kill:~/Downloads/214 Systems Programming/pa2$ make clean

To use this Makefile, simply navigate to the directory containing the Makefile and run the command make. This will build the myshell 
executable. To remove all files created during the build process, run the command make clean.
------------------------------------------------------------------------------------------------------------------------------------------
USAGE : Testing strategy and test cases.
------------------------------------------------------------------------------------------------------------------------------------------
This program mysh takes up to one argument. When given one argument, it will run in batch mode. When given no arguments, it will run in 
interactive mode. [Note : The file myscript.sh contains our test cases to check for batch mode.]

Batch mode : When called with an argument, mysh will open the specified file and interpret its contents as sequence of commands. The 
command are given by lines of text, separated by newlines. mysh will execute each command as soon as it is complete, before proceeding 
to execute the next command. mysh terminates once it reaches the end of the input file or encounters the command exit.

am3002@kill:~/Downloads/214 Systems Programming/pa2$ cat myscript.sh
am3002@kill:~/Downloads/214 Systems Programming/pa2$ ./myshell myscript.sh

Interactive mode : When called with no arguments, mysh will read commands from standard input. Before reading a command, mysh will write a 
prompt to standard output. After executing the command, mysh will print a new prompt and read the next command.

am3002@kill:~/Downloads/214 Systems Programming/pa2$ ./myshell
Welcome to my shell!
mysh> echo Hello, world!
------------------------------------------------------------------------------------------------------------------------------------------
Test Case Example 1 : mysh> foo bar < baz | quux *.txt > spam

In such a case, the final output of the command will be saved in the "spam" file. The specific contents of the output will depend on the 
behavior of the "quux" program and the contents of the ".txt" files that match the wildcard pattern.

Test Case Example 2 : mysh> echo foo    bar        >      baz

In such a case, the final output of the command will be saved in the "baz" file. The command tokens can be separated by multiple spaces.
This program ignores leading and trailing whitespace. Therefore, "foo bar" is written to "baz" file.

Test Case Example 3 : mysh> ec*o foo

In such a case, if working directory has an echo file, "ec*o foo" will call the echo command that found in the six specified directories. 
On the other hand, if working directory doesn't have an echo file, "ec*o foo" will do nothing and print error.

Test Case Example 4 : mysh> echo foo bar > baz (OR) mysh> echo foo > baz bar (OR) mysh> echo > baz foo bar

Test Case Example 5 : mysh> bar < foo > baz (OR) mysh> bar > baz < foo

All these cases are eqivalent to each other, as input/output redirection can occur anywhere in a command string. This program takes care
of such cases and the final output of the command will be saved in the "baz" file.

Test Case Example 6 : mysh> echo foo > bar | baz (OR) mysh> echo foo > baz > bar

In such a case, the final output of the command will be saved in the "bar" file.

Test Case Example 7 : mysh> pwd > temp

In such a case, the final output of the command will be saved in the "temp" file.

Test Case Example 7 : mysh> pwd | ./revline

In such a case, the pwd command will print the current working directory to the standard output, and the '|' (pipe) symbol will redirect 
the output of the pwd command as input to the revline command. If there is no such file, the program gives an error.

Test Case Example 8 : mysh> touch foo | exit (OR) mysh> exit | echo bar

In the first case, the program first creates the file "foo" and then process to exit. While, in second case, the program exits directly.
------------------------------------------------------------------------------------------------------------------------------------------