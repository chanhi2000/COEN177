# lab01b

## THE SECOND PROGRAMMING TASK
You are to write a program that will create exactly seven child processes (not including the initial program itself). You are not to allow any single process to create more than two child processes.

```c
#include <unistd.h>			// declares fork(), getpid(), getppid()
#include <sys/types.h>		// declares pid_t 
#include <sys/wait.h>		// declares waitpid()
#include <stdio.h>          

#define MAXPROC 7			// maximum # of processes
#define CHILD 0				// child indicator (NOT parent)

void  demo(int ProcNum)
{
		pid_t pid; 
		int status;
		
		if (ProcNum < MAXPROC) 
		{
				pid = fork();
				if (pid != CHILD) {
						//parent
						printf("parent: %d\n", getpid());    
						waitpid(-1, &status, 0);
				
				} else {
						//child
						printf("child: %d, parent: %d\n", getpid(), getppid());
						demo(ProcNum+1);
						return;
				}
		}
}

/*
 * prints the process IDs of one parent and seven child processes
 * the solution tree is as follows:
 * 
 *					parent
 *					  |
 *					  a   
 *					  |
 *					  b
 *					  |
 *					  c
 *					  |
 *					  d
 *					  |
 *					  e
 *					  |
 *					  f
 *					  |
 *					  g
 */

int main(void) 
{
		demo(0);
}
```