# Profiling-of-Thread-Scheduling-in-Linux-Kernel

Compile and run mythreads.c to create multiple processes and threads
gcc -o mythread mythreads.c -lpthread

Write two systems calls sys_start_pid_record() and sys_stop_pid_record(buff). Start system call starts recording of jiffies, processes and threads in buffer. Stop system call stops the recording and copy the contents of buffer to user buffer. Changes made in core.c file. Following code added to record the activities:

//Structure to record thread and process ID

typedef struct{
	unsigned long long myJifi;
	pid_t processId;
	pid_t threadId;
}recordProcThread;

recordProcThread *record=NULL;

int flag=0;
int bufIndex;


//Start system call which starts recording of threads and proces IDs in buffer
asmlinkage int sys_start_pid_record(void)
{
	bufIndex = 0;
	if((record = kmalloc(sizeof(recordProcThread[10000]), GFP_KERNEL)) == NULL)
	{
		printk(KERN_ALERT "Error: Not able to allocate memory using kmalloc\n");
		return -EFAULT;
	}
	flag = 1;
	return 0;
}

//Stop system call which stops recording of threads and process IDs in buffer
asmlinkage int sys_stop_pid_record(void* __user user_buffer)
{
	flag=0;
	if((copy_to_user(user_buffer, record, sizeof(recordProcThread[10000]))) < 0)
	{
		printk(KERN_ALERT "Error in copy_to_user");
		return -EFAULT;
	}	
	
	Following code added in static void __sched __schedule function to record the jiffies, threads and processes.
//Untill stop system call invokes and buffer size is less than MAX size
if(flag && bufindex < 10000) {
   record[bufindex].myJifi = jiffies; record[bufindex].processId = next->pid; record[bufindex].threadId = next->tgid; bufindex++;
}

Now print the buffer and observe the thread selection sequence. If the next selected thread by cpu belong to different processes, then its global scheduling. If the next selected thread by cpu belong to same process, then its local scheduling. 
