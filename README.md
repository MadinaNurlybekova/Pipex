# ft_pipex

Your program will be executed as follows:
**./pipex file1 cmd1 cmd2 file2**

It must take 4 arguments:
  - file1 and file2 are file names.
  - cmd1 and cmd2 are shell commands with their parameters.

It must behave exactly the same as the shell command below:
**$> < file1 cmd1 | cmd2 > file2**


## Step 1. 

1. Create and initialize t_pipex struct that will store following information:

```C
typedef struct s_pipex
{
	int		fd_pipes_count; // amount of FDs for all pipes (cmd_count - 1) * 2
	int		cmd_count; // number of commands
	int		infile_fd;
	int		outfile_fd;
	int		*fd_pipes; // array of pipes' FDs
	int		fd_ind; // index of a pipe
	int		is_heredoc;
	char	*infile_name;
	char	*outfile_name;
	char	*limiter;
}			t_pipex;
```

2. Create pipes. (using fd_pipes_count):

```C
int	create_pipes(t_pipex *pipex)
{
	int	i;

	i = 0;
	while (i < pipex->fd_pipes_count)
	{
		if (pipe(pipex->fd_pipes + i) == -1)
		{
			free(pipex->fd_pipes);
			return (-1);
		}
		i += 2;
	}
	return (1);
}
```
3. Create list of commands together with parameters;
4. Execute command:
     - we do fork() for every command --> while (--cmd_count)
         and increment fd_ind by 2 --> fd_ind+=2;
     - here we have separate do_fork functions for first command, middle commands, and last command;
5. Forking:
   - we fork and go into the child process (fork() == 0).
   - inside child process we redirect the input and output using ft_dup2(input_file_fd, output_file_fd) function:
   ```C
   int	ft_dup2(int fd1, int fd2)
    {
      if (dup2(fd1, STDIN_FILENO) == -1)
        return (-1);
      if (dup2(fd2, STDOUT_FILENO) == -1)
        return (-1);
      return (1);
    }
   ```
  - after the redirection, we close all the open FDs inside the child process:
```C
void	close_fd(t_pipex *pipex)
{
	int	i;

	i = 0;
	while (i < pipex->fd_pipes_count)
	{
		close(pipex->fd_pipes[i]);
		i++;
	}
}
```
  - and then we execute the command itself with execve still inside the child process.
6. After the execution of all the commands (while (--cmd_count) --> cmd_count == 0), we are back to the parent process:
  - close all pipes in parent process:
    ```C
    void	ft_close(t_pipex *pipex)
    {
    	close_fd(pipex);
    	if (pipex->infile_name)
    		close(pipex->infile_fd);
    	close(pipex->outfile_fd);
    }
    ```
 - wait(&status) - waiting for the child processes to end.
 - free memory;
