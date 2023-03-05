# ADB System Services

From command line we can interact with system services. For example, we can print a list of available system services. On Android 10 Pixel emulator there are 165 services, which we can compare to only 65 on analogous Pixel 3 XL emulator but with Android 4.1. A lot has changed over the years.

<img width="400" alt="Screenshot 2023-03-04 at 6 51 22 PM" src="https://user-images.githubusercontent.com/70295997/222950379-51d9bb94-6169-4277-b87e-bb0f829e0bff.png">

Some system services that we can interact with are the ActivityManager and PackageManager service. We can even interact with the StatusBarService which has its own system service.

<img width="400" alt="Screenshot 2023-03-04 at 10 43 56 PM" src="https://user-images.githubusercontent.com/70295997/222950407-d4261e93-fe3b-4b4a-97b8-36332ed73c74.png">

Through command-line we can trigger binder transactions. Binder is an IPC (Inter Process Communication) mechanism, which is used for communication between system services and app and user apps as well.  And we can invoke the following commands to show and hide the status bar:

    service call status bar 1 (show the status bar)
    service call status bar 2 (hide the status bar)

<img width="400" alt="Screenshot 2023-03-04 at 8 26 58 PM" src="https://user-images.githubusercontent.com/70295997/222950485-8ae37206-940a-46f7-a5db-6f43d2468dec.png">
<img width="400" alt="Screenshot 2023-03-04 at 8 20 24 PM" src="https://user-images.githubusercontent.com/70295997/222950513-f3364e3e-88d8-4ae2-9423-888a16402403.png">

We can interact this way with many different services.

If we are really curious about what’s going on our Android device, we can invoke Process Status and show All. An Interesting thing that we can find here is <code>init</code>. As theory says, <code>init</code> is the 1st process on an Android device, and indeed it is the 1st because it has process id 1. And <code>zygote</code>,  which is the root process for all Java apps, has a certain process id. And of the Java apps that are running have the parent process id of <code>zygote</code>.

<img width="600" alt="Screenshot 2023-03-04 at 8 34 06 PM" src="https://user-images.githubusercontent.com/70295997/222950627-209c5489-fad4-4dd4-b682-7dcc4f0844ed.png">

    stop zygote
    start zygote

To play a little bit with <code>zygote</code>, we can start or stop it. This triggers the whole user space to reboot. It’s not a complete reboot of our device, because we are basically stopping the parent process and also stopping all the child processes, and all Java spaces get restarted. And because kernel has a special check for whether <code>zygote</code> is running or not, it instantly restarts it.

System Services are organized in a way that they are exposed to something called System Server. And System Server exposes services like Activity Manager, Package Manager, etc.

<img width="400" alt="Screenshot 2023-03-04 at 8 41 59 PM" src="https://user-images.githubusercontent.com/70295997/222950715-947ce22b-7620-48da-b596-466791a8fcc2.png">

We cannot find this, if we try to look at running processes and search for something related to Activity Manager or Package Manager - these return no results. But instead we find something called System Server, which has the same parent process as <code>zygote</code>. 

<img width="600" alt="Screenshot 2023-03-04 at 8 43 49 PM" src="https://user-images.githubusercontent.com/70295997/222950744-9cde7c99-ed8a-4289-b571-5ef31353f50c.png">

And these System Services are exposed through System Server. So they are all running in the same process.

The interesting thing is that if we look at threads, we might find out that System Server runs on 132 threads. So all the 165 system services are running on 132 threads.

<img width="600" alt="Screenshot 2023-03-04 at 8 47 33 PM" src="https://user-images.githubusercontent.com/70295997/222950768-9de5fb63-1517-4e2a-80c0-464abf49a476.png">

Also when we take a closer look, Activity Manager tuns on 3 threads and Package Manager runs on 2 threads.

<img width="200" alt="Screenshot 2023-03-04 at 8 50 17 PM" src="https://user-images.githubusercontent.com/70295997/222950833-addfc436-1b50-4750-807f-52eefa3f21da.png">

A simple hello_world app needs to run at least 13 threads. Why? First of all, it has the <code>main</code> thread.

<img width="600" alt="Screenshot 2023-03-04 at 8 53 15 PM" src="https://user-images.githubusercontent.com/70295997/222950906-8548513f-e372-423e-a85a-5eb6c075d99f.png">

We are essentially busting a myth. Because there’s a myth that Android apps run on a single thread. Not true, they run on at least 13. We have threads related to debugger, garbage collection, finalizing and reclaiming memory. We have something called RenderThread which renders our UI. And we have some open Binder threads which are used for communication, for example, with Activity Manager. They have their own threads, on which if they receive a call to start another activity. Then this call is redirected to the <code>main</code> thread, and then another activity is started on the <code>main</code> thread of the given app. 
Essentially, it’s not true that Android apps by default are single-threaded, they are 13-threaded at least.

And we can create a bunch of other threads, in this case 5 new threads.

<img width="600" alt="Screenshot 2023-03-04 at 8 59 52 PM" src="https://user-images.githubusercontent.com/70295997/222951030-7b1239fe-9e0c-465d-86bb-c35119b1da86.png">

We can see them in this listing as well.

<img width="600" alt="Screenshot 2023-03-04 at 9 00 33 PM" src="https://user-images.githubusercontent.com/70295997/222951043-d123f305-e7a1-4b47-99e4-9147103e996f.png">

It’s a common misconception that Java threads are somewhat different than Linux system threads. They are not, they are just some handy abstraction of system threads. But in fact what’s going on is that the system created all these threads, not Java Virtual Machine.

And we can also see all these threads in a tab that perhaps not too many people look at. 
When we debug our app, we can see all these threads when they are running. And we can stop/resume them one by one. I’ve discovered recently that we can change values of variables when we are debugging.

<img width="600" alt="Screenshot 2023-03-04 at 9 03 48 PM" src="https://user-images.githubusercontent.com/70295997/222951179-b74155f2-c6e5-458d-a785-e39f9e3b16b4.png">

















