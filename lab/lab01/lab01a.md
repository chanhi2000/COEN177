# lab01a

## THE BASIC PROGRAMMING TASK
You are to write a very simple shell program, similar to the example covered in lecture. The shell should terminate if the user enters “exit” at the command line, and should attempt to launch a program in response to any other input string. You do not have to parse any parameters, nor do you need to support background execution of processes. The simplest possible shell to demonstrate is all that is required. You may wish to write a “hello world” program, so that you have an executable file that you could launch from your shell.

```c
#include <unistd.h>				// declares execve()
#include <sys/types.h>			// declares pid_t
#include <stdio.h>				// declares printf(), fgets()
#include <sys/wait.h>			// declares waitpid()
#include <stdlib.h>				// declares exit()
#include <string.h>				// declares memset, strchr(), strcmp()

#define TRUE 1
#define MAXCMDLEN 20			// maximum command length
#define CHILD 0					// child process indicator (!parent)


char command[MAXCMDLEN];
char *parameters[1];			// ignore parameters

// print_prompt: prints shell prompt for input
void print_prompt(void)
{
		printf(">> ");
}

// read_command: reads input from stdin into command
void read_command(char command[])
{
		char *p;

		// clear array
		memset(command, '\0', MAXCMDLEN);

		// exit parent process on empty input
		if (fgets(command, MAXCMDLEN, stdin) == NULL) {
				exit(0);
		} else {
				p = strchr(command, '\n');
				*p = '\0';
				if (strcmp(command, "exit") == 0) {
						exit(0);
				}
		}
}


int main(void)
{
		pid_t pid;
		int retval;
		int status;

		while (TRUE) {
				print_prompt();
				read_command(command);

				pid = fork();
				if (pid != CHILD) {
						// parent process
						waitpid(pid, &status, 0);
				} else {
						// child proccess
						retval = execve(command, ignore, 0);
						if (retval < 0)
								printf("command error: %s\n", command);
						exit(retval);
				}
		}
}
```
