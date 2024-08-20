# Profiling and Progress

Profiling and progress are middlewares that both depend on the task registry middleware.

## Task Registry

The structure of VerCors execution is tracked as a tree of tasks. Each parent task can have any number of sub-tasks, where the parent task cannot end before the subtask. It is allowed and expected that a parent task can have multiple active subtasks in the case that the subtasks are executed concurrently. Tasks can be as fine-grained or course-grained as you like, but there is significant to creating a task on the VerCors level, so consider only making new tasks e.g. around process interaction.

Each task defines explicitly what its supertask is by overriding `def superTask: AbstractTask`. In most cases there are utility methods in the `hre.progress.Progress` object that grab the current task as the parent for the subtask that is being entered, but if it is necessary to obtain the current task, it is available at `hre.progress.TaskRegistry.currentTaskInThread`. Note that the current task is stored in a `ThreadLocal`, meaning that while every thread individually has its own correct notion of the current task, the task is not accounted as the current task in any further started threads. In this case the task either has to be obtained manually before starting a new thread, or you let a shorthand deal with this for you in the `Progress` object. In particular it is supported to put parallel collections in `Progress.map`, `Progress.foreach`, etc.

Each task can optionally define a weight, which is used to increase the amount of progress done in the parent task. Note that each task stores a number between `0.0` and `1.0` to recall how far along we are, so the sum of `progressWeight` of subtasks must not exceed `1.0`. You can define the weight of a task by overriding `def progressWeight: Option[Double]`.

Lastly each task must define `profilingBreadcrumb` and `renderHere`, which are used respectively in profiling and in progress rendering.

## Profiling

VerCors has an internal capability to profile itself. Typically profiles trace the call stack, but we trace the tasks in the task registry. This is very helpful, because it enables us to do things like print the precise verification goal as a "stack" entry. The output format of a vercors profile is [pprof](https://github.com/google/pprof), see the wiki for usage information.

On supported platforms we use the `getrusage` syscall, which is supported in at least Linux kernels and macOS. MacOS does not support a feature where usage is also tracked for child processes for which we have called `wait`, so precise usage for `z3` is not tracked on MacOS unfortunately. For windows there is no support other than just tracking wall time, though conceivably we could look into using `GetProcessTimes`.

The labels used in the stack of the profile are simply the trail of `profilingBreadcrumb` values towards the task that is being polled. Note that although the method that updates the profile is called `poll`, it is actually only invoked exactly at the start and the end of a task: there is no timer that polls tasks on an interval.

## Progress

Although VerCors does print a progress bar, what we really mean by progress is the principle that we should have some idea what the tool is doing at all times. Ideally we are headed for a future where verifying a file is extermely performant, and knowing what the tool is doing at a given moment becomes irrelevant, but in the meantime we have to deal with the reality that verifying programs can be minutes- or hours-slow for nearly inscrutable reasons.

To make things a bit more bearable, each task in the task registry offers a `ProgressRender` when `renderHere` is called. This is simply a list of text lines, together with a line index: the anchor to which we should point. In particular tasks that relate to a point in the input print a message next to a sliver of the source, where the message is then the anchor line. A good approach to diagnose a slowness in VerCors is to make tasks more fine-grained, which helps users and developers in the future. A recent example of this is that it seemed that verification was slow to "start up," so now VerCors reports what it is working on just before starting verification (translation, diagnostic output, consistency checking or idempotency checking). You do not need to immediately understand what that means, but you can imagine it is helpful to receive a bug report "tool stuck on idempotency checking" rather that "tool stuck in verification."

A standing problem is that the performance of task accounting in the backend is bad: in unlucky cases we can spend up to 50% of time just shuffling around tasks about the verification, rather than working on the verification. Nevertheless it has proven very useful to get an idea about what tasks take long, whether there is a large number of tasks or a large number of branches, what path conditions are particularly costly for a verification goal, etc. Note also that a large accounting overhead is indicative of a problem in and of itself: it implies that the number of different verification goals is very large, so the time-saving opportunity is most likely in the direction of reducing the amount of verification goals (or branches under which they are checked) anyway.

