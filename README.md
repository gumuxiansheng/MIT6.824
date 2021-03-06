# MIT 6.824 - Spring 2020

This is a notebook on study MIT 6.284.

**About the lab:** The source code will be private, however, some thoughts may be inoffensive.

## [Lab 1: MapReduce][lab1]

This lab should build a MapReduce system, which may like below:
![MapReduce](https://github.com/gumuxiansheng/MIT6.824Labs/blob/master/docImg/2020-04-18-16-43-48.png?raw=true)
*ImageSource: Dean, J., & Ghemawat, S. (2008). MapReduce: Simplified data processing on large clusters. Communications of the ACM, 51(1), 107-113. https://doi.org/10.1145/1327452.1327492*

The key is to complete the code for master and worker. While running, the master should keep trace of all the tasks and assign a proper one to the worker when requested. The tasks should be "map" task or "reduce" task, and "reduce" task should be run after all the "map" tasks finished.

In this lab, we can initial the master with *X* "map" tasks and *Y* "reduce" tasks while *X* is the number of the input files, that is, the number of the pg-\*.txt files(8 in the lab), and *Y* is determined by the User Program and passed into master, called nReduce(10 in the lab). All the "map" and "reduce" tasks are stored in the *TasksMonitor*. We should get $X+Y$ tasks. While the master is running, it listens for RPCs from workers. 

If one of the workers request a task through RPC, the master will check the TasksMonitor if there is a proper task for this worker currently. The master should check the unassigned or timeout "map" tasks first and only after all the "map" tasks are done should the master start to assign "reduce" tasks. If the worker request a task but all "map" tasks are assigned and no "reduce" tasks can be started, the master will assign a "wait" task to the worker and the worker will sleep for a second then request again. If all the tasks in the TasksMonitor are done, the master will send a "exit" task back to the worker and the worker will exit. The master will periodically check if all the tasks are done, if so, the master will exit and if the worker found nowhere to connect to the server, it would consider the tasks are all done and will exit automatically.

While the worker received a "map" task, it reads all the file contents and split all words and store the word's key-value like {"Key":"Additional","Value":"1"} in *mr-X-Y.json* file, where X is the "map" task number and Y is the calculated "reduce" task number through *ihash* method word by word.

If the worker received a "reduce" task, it gets a "reduce" task intermediate files generated by "map" tasks belong to this task, named "mr-\*-Y.json" where Y is the "reduce" task number. So this worker will read all the key-values in all these files and reduce them into output files named "mr-out-Y".

When the worker finished a task, it will start to request for an other task if the prior task is not "exit", at the same time, it should tell the master the prior task is done so the master can update the status of the task.

At last, we got $X \times Y$ intermediate json files for "reduce" and nReduce reduce out files.

In this lab, we should be careful when reading and writing *TasksMonitor* in master processes. It's common to add a mutex lock on it.

The framework of the lab is like below:

**The master:**
```go
// DistributeTask to distribute a task on a worker's request.
func (m *Master) DistributeTask(args *TaskArgs, reply *TaskReply) error {
    // TODO:
    // 1. Update the sendback task's status
    // 2. Assign a proper task to the current worker

    return nil
}

//
// create a Master.
// main/mrmaster.go calls this function.
// nReduce is the number of reduce tasks to use.
//
func MakeMaster(files []string, nReduce int) *Master {
    m := Master{}

    // TODO: Initial tasks.

    m.server()
    return &m
}
```

**The worker:**
``` go
func Worker(mapf func(string, string) []KeyValue,
    reducef func(string, []string) string) {

    // TODO: Dummy task to store the just finished task. Initialized by empty task.

    for {
        // TODO: Ask the master for a task

        if receivedTask.TaskType == WAIT {
            // TODO: Sleep for a second
        } else if receivedTask.TaskType == EXIT {
            // TODO: Exit
        } else if receivedTask.TaskType == MAP {
            // TODO:
            // 1. Do map task
            // 2. Change task status to done for sending back to master
        } else if receivedTask.TaskType == REDUCE {
            // TODO:
            // 1. Do reduce task
            // 2. Change task status to done for sending back to master
        }
    }

}
```

[lab1]:https://pdos.csail.mit.edu/6.824/labs/lab-mr.html
