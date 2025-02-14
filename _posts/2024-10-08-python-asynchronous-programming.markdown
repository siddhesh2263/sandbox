---
layout: post
title:  "Python - Asynchronous Programming"
date:   2024-10-08 10:02:29 -0400
categories: jekyll update
---


## Part 1 - Threading Module

Click [here][repo-link-threading] for code repository.

Threads wouldn't be ideal where a lot of computation is involved. If there are operations that involve image resizing, etc., threading would not speed up the process by that much. That would be an example of something that is CPU bound, and not IO bound. In those scenarios, threads can slow down the script, because threads involve some overhead when they are created and destroyed. So, given the case, we need to decide whether we need to go with threading, or multiprocessing.

<br/>

The below image shows usage of threads using Python's inbuilt 'threading' module. The `do_something` function is passed to the target without parenthesis, followed by the function arguments. The `start()` function is called to create the tread and start execution. The `join()` function is used to stop the code flow till all the threads are executed. However, this `join()` function cannot be added at the end of the `start()` function at the end of each iteration, since it would behave same as synchronous operation. To overcome this, the threads are stored in a list, and are checked if their execution is completed in a different loop.

![image tooltip here]({{ "/assets/python_threading/001_thread_direct_use.png" | relative_url }})

<br/>

Another way to manage threading is using the Python's inbuilt `concurrent.futures` module. A context manager for the `ThreadPoolExecutor` is created. The below image shows the usage of list comprehension, in which the context name (in this case named as `executor`) takes in the function to be executed using threads, and the arguments for that function. The outcome is stored in the `results` variable. Although not necessary, but if we want to observe if the threads are executed, the `concurrent.futures.as_completed()` function will print out the executed threads from the `results` variable, which has stored all the threads.

![image tooltip here]({{ "/assets/python_threading/002_futures.png" | relative_url }})

<br/>

The threads in the above code were executed using a loop. The same can be done using the `map` function. The function name and its arguments are passed to the `ThreadPoolExecutor`'s context name.

![image tooltip here]({{ "/assets/python_threading/003_using_map.png" | relative_url }})

<br/>

We now see an example where images are downloaded from the internet. The `download_image` function downloads the images. Using the `concurrent.futures` module, we pass in the function and the list of image URLs. The time taken to download some 15 images drops down from `4.8` seconds to `1.1` seconds. The performance change would be larger if more images were involved.

![image tooltip here]({{ "/assets/python_threading/004_using_threads_image.png" | relative_url }})

<br/>

<br/>

## Part 2 - Multiprocessing Module

Click [here][repo-link-multiprocessing] for code repository.

We'll be using Python's inbuilt `multiprocessing` module to work with processes.

The below image shows the creation of two processes. The function to be run in parallel is passed as the target. The `start()` function begins the process. The `join()` function makes sure to halt code flow until the processes are executed, and then move on to the next line.

![image tooltip here]({{ "/assets/python_multiprocessing/001_direct_use.png" | relative_url }})

<br/>

The below error is raised when running the code on a Windows machine.

![image tooltip here]({{ "/assets/python_multiprocessing/002_error_main.png" | relative_url }})

<br/>

To rectify this error, the code is moved into the `main` function. This is specific to Windows OS.

![image tooltip here]({{ "/assets/python_multiprocessing/003_error_rectify.png" | relative_url }})

<br/>

Multiple processes can be started using a loop. The `join()` function call cannnot be added after starting each process, since it would be same as running the functions synchronously. Instead, all processes are appended in a list. In another loop, the `join()` function is used for waiting for each process execution.

![image tooltip here]({{ "/assets/python_multiprocessing/004_multiple_processes.png" | relative_url }})

<br/>

The below code shows how arguments can be passed to the function.

![image tooltip here]({{ "/assets/python_multiprocessing/005_passing_arguments.png" | relative_url }})

<br/>

Another way to begin a process, is by using the `concurrent.futures` module. A context name, in this case the `executor` handles the processes. The `submit()` function schedules the target function to be run.

![image tooltip here]({{ "/assets/python_multiprocessing/006_futures.png" | relative_url }})

<br/>

The below code shows how list comprehension can be used to execute multiple processes. The `as_completed()` function is used to check the status of the process, in this case it prints out the function's return value when the process ends. This is done using the `result()` function.

![image tooltip here]({{ "/assets/python_multiprocessing/007_multiple_futures.png" | relative_url }})

<br/>

The below code shows how a list of different arguments can be passed to a function, which is to be run using different processes.

![image tooltip here]({{ "/assets/python_multiprocessing/008_passing_list.png" | relative_url }})

<br/>

The `map()` function maps the arguments with the target function. The behaviour is same as any loop or list comprehension syntax.

![image tooltip here]({{ "/assets/python_multiprocessing/009_map_function.png" | relative_url }})

<br/>

The goal is to reduce processing time of images. The `process_image()` function performs some operation on the image. The `concurrent.futures.ProcessPoolExecutor` function is used for executing the processing of these images in parallel. The results show a decrease in the processing time when the multiprocessing module is used.

![image tooltip here]({{ "/assets/python_multiprocessing/010_images.png" | relative_url }})

<br/>

We cannot be sure if the time taken is IO bound or CPU bound. This is because the images are read and copied to another folder, so we are not entirely sure if the `filter()` function is the most resource consuming. This can be tested for a larger batch of images.

Use benchmarks to determine if threading or multiprocessing is a better fit.

Use threads for activities that are IO bound. Use processed for activities that are CPU bound.

Even if there are more processes than cores, it was still able to finish in lesser time. Why?

[repo-link-threading]: https://github.com/siddhesh2263/corey_threading
[repo-link-multiprocessing]: https://github.com/siddhesh2263/corey_multiprocessing