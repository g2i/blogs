# Profiling React Native App's Runtime Performance Using Hermes

### Debug like a detective. This article sums up how React Native works and the App's Runtime Overview Performance Using Hermes.

![](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f88b1388c10d6ec6f029b27_header_img.png)

We have a tool to help you solve those mysteries faster so you can hang up your Sherlock Holmes hat, and get back to writing code.

[Hermes-profile-transformer](https://www.npmjs.com/package/hermes-profile-transformer)  allows you to visualize your app's performance in an easy and accurate way.

Before diving into what the tool does, let's make sure you know what profiling is when it comes to applications.  [Profiling](https://en.wikipedia.org/wiki/Profiling_(computer_programming))  is "a form of dynamic program analysis that measures, for example, the space (memory) or time complexity of a program, the usage of particular instructions, or the frequency and duration of function calls."

Simply put - profiling is incredibly important in understanding the runtime performance of your program, or in particular React Native app, and finding solutions to optimize that runtime.

Hermes-profile-transformer helps developers profile and visualize the performance of JavaScript running on  [Hermes](https://github.com/facebook/hermes)  in a React Native app. A common way to analyze the performance of a program is to analyze the sampling profile in the  [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools)Performance tab.

Here's where we run into a roadblock. The profiler output from Hermes comes in a different format than the Chrome DevTools processes. Those two formats are like fire and ice. It's difficult to get them to pair well.

This is where our tool comes in. It transforms the Hermes profiler output into a format that Chrome's event tracing can handle. This allows developers to use Chrome DevTools to visualize their app's sampling profile and understand which functions are taking a long time to complete.

Let's dive into profiling your React Native app with Hermes and visualizing the app's performance.

## **What is Hermes?**

Hermes is an open-source JavaScript engine designed to run React Native applications on Android. Hermes stands out amongst its competitors because of its performance when it comes to start-up times, memory usage, and app size.

The use of Hermes in React Native applications isn't required. However, considering that Hermes has been explicitly designed by Facebook for React Native, it is bound to be the most commonly adopted mode of running React Native applications in the future.

For more information on Hermes and how to use it, you can check out this helpful  [documentation](https://reactnative.dev/docs/hermes).

The Hermes engine bolsters the performance of the React Native application in a mobile environment. You can see examples of this in the experiments performed by  [Ram](https://twitter.com/nparashuram). These demonstrations proved how powerful and fast Hermes can be when used in the React Native ecosystem.

![Hermes Performance Improvements](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd91206f535d549230_Zkbbd4HAWYI-boTHnnF6qZEJnJEcK3SZbZ1_ki3ygKQkdBUKyHvW9KRTNJsgKFkQHCqXI7xzu5G7cngbw_IvGsSKWGUrRt4zeZS6t9trAOJvkRKGO3UFaelMBX-0cj0cmMcyLPer.jpeg)

You can dive deeper into Ram's excellent Hermes experiments  [right here](https://blog.nparashuram.com/2019/07/facebook-announced-hermes-new.html).

### Trace Events Formats

React Native applications running on Hermes have a mechanism in place to profile the application as it runs. The process to do this is relatively simple. The developer menu of RN applications running on Hermes has an option to toggle profiling. Upon enabling profiling, the profiler starts to create trace events (in the form of samples and stack frames) which can be used to obtain timing information of the functions running.

However, the trace events created by the Hermes profiler are of the JSON Object format as opposed to the Chrome supported JSON Array Format. So, we need a  [transformer](https://www.npmjs.com/package/hermes-profile-transformer)that converts the Hermes profile into JSON Array Format which is supported by Chrome DevTools.

### Understanding Trace Events

Trace events are data points that determine the state of an app at any point of time. These data points include names of the functions running, their process and thread IDs among many others. The most common sampling profiles can usually be parsed as JSON and can be in 2 formats:

**JSON Array Format**  - This is the simplest format. It uses an array of event objects which indicate the start and end times (based on phases) and can be loaded into Chrome DevTools to visualize the performance of an application. For example:

```json
[
{ name: "Asub", cat: "PERF", ph: "B", pid: 22630, tid: 22630, ts: 829 },
{ name: "Asub", cat: "PERF", ph: "E", pid: 22630, tid: 22630, ts: 833 },
];
```

The JSON array format is possibly the simplest and most effective way to store the profiling information of an application. It is easy to read and hence is widely adopted.

**JSON Object Format**  - JSON Object format, as its name suggests, is a collection of key-value pairs that can be used to capture the state of the application. For example:

```json
{    "traceEvents": [
     {"name": "Asub", "cat": "PERF", "ph": "B", "pid": 22630, "tid": 22630, "ts": 829},
     {"name": "Asub", "cat": "PERF", "ph": "E", "pid": 22630, "tid": 22630, "ts": 833}
   ],
   "displayTimeUnit": "ns",
   "systemTraceEvents": "SystemTraceData",
   "otherData": {
     "version": "My Application v1.0"
   },
   "stackFrames": {...}
   "samples": [...],
 }
```

The object contains 3 keys, namely,  `traceEvents`,  `samples`, and  `stackFrames`, each of these are extremely important to analyze the performance of the application. The benefit of this structure over the JSON Array Format is that the information is stored in a more efficient format, allowing you to add additional data points in the future.

In the case of the Hermes profile, we get a JSON Object back, with metadata events in its  `traceEvents`  property and the samples and  `stackFrames`  properties capturing information about all the other functions running during the profiling interval. These properties contain essential information such as event categories, phases, and timestamps that help us gather a ton of information about our React Native application.

### Events & Phases

Events are actions that are triggered while an application is running. These events can be of different types and can have a variety of phases. The simplest type of event is the Duration Event. As the name indicates, these events simply capture and log the time it takes to complete an action. Two distinct data points, namely the  `start`  and the  `end`  of a duration event, specify the status of these events. These data points are known as phases.

![](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f877cca6a6e5b56035138c3_Screen%20Shot%202020-10-14%20at%203.33.24%20PM.png)

For our particular use case, the events featured above are just function calls. Different samples indicate when a function is pushed and popped from the call stack. The  _phases_of events are the most important properties for tracing events. Why? Because they indicate the state of the functions in the call stack. We can assume that the Hermes profile consists of only Duration Events.

## **Interpretation of Different Events in a Profile**

The Hermes profile is of the JSON Object Format, however, the  `traceEvents`property does not contain timing information of functions as we expect. Instead, the  `traceEvents`  property merely contains certain metadata events that do not provide information about app performance. This information is captured in the samples and  `stackFrames`  properties.

The  `samples`, as the name suggests, consists of snapshots of the function call stack at specific timestamps. They also contain an sf property that corresponds to an element in the  `stackFrames`  property of the profile.

![](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd6573ac25e439d21a_JUaQ26QqK5jYK6WSeXFFk-MNTIqQhq13YaMJ9R2kOyOAdfuptryD8kX1P4QFwVl0kbLfUJ0o_4fvGFDUtk-ygQqqtRGV0uvheyTKDrdFRkAPtQkYYoHyYPJTi7kHEmIpKA4RDQCv.png)

The events can also be filed into broad categories based on their origin. Events that are usually written in source code fall under the category of Javascript, while the events that occur natively on the platform are appropriately categorized as Native. Metadata is another primary category of events and is usually collected in the  `traceEvents`property of the Hermes profile.

### Definitions

1.  Nodes - Nodes stand for all the events possible in a function call stack. For example:

```json
"2" : {
"line": "8",
"column": "12",
"funcLine": "8",
"funcColumn": "12",
"name": "f1(test.js:8:12)",
"category": "JavaScript",
"parent": 1
}
```

Each of these stack frame objects can have corresponding beginning and ending events. The individual begin/end events can be referred to as nodes. Nodes only have context in particular timestamps, i.e. a node can be in  `start`,  `end`  or  `running`  state at a particular timestamp.

2.  Active & Last Active Nodes - Building upon the previous definition, the active nodes at any particular time are the nodes that are active in the current timestamp. The Last active nodes similarly stand for the nodes that were active in the preceding sampling time.

3.  Start & End Nodes - We can define start nodes as nodes active in current timestamp but absent in last active nodes, while the end nodes can be defined as nodes in last active nodes but absent in current active nodes.

![Start and end nodes](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd24eace8ffdfe96fe_0ZVYDTKbdwc1rIqnI-vCQ7DK8oTu-m7T9a3czeUPEY7XJKu90975J8irvqL2lH4Y0jgjiY7FLaYOTdegN8hXkQhxuyo0bQzs8vy2hd1r6kWoLDG5UUsBgAqVH_lzDwGyKzO5ucRr.png)

The Hermes profile transformer works by identifying start and end nodes at each timestamp and creating events from these nodes to be displayed on Chrome DevTools.

## **Usage: With React Native CLI**

We also implemented a new command  `react-native profile-hermes`  on the  [React Native CLI](https://github.com/react-native-community/cli)  to make the process smooth for developers. The command automatically transforms the profile using our  `hermes-profile-transformer`package and pulls the converted device to the user's local machine.  Follow these steps to start profiling your application:

**Step 1**‍

First, you need to enable Hermes in your React Native app by following these  [instructions](https://reactnative.dev/docs/hermes).

**Step 2**

Record a Hermes sampling profiler by following these steps:

-   Navigate to your running Metro server terminal.
-   Press d to open the Developer Menu.
-   Select Enable Sampling Profiler.
-   Execute your JavaScript in your app by pressing any button.
-   Open the Developer Menu by pressing d again.
-   Select Disable Sampling Profiler to stop recording and save the sampling profiler. A toast will show the location where the sampling profiler has been saved, usually in /data/user/0/com.appName/cache/*.cpuprofile

![Toast Notification of Profile saving](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd5e3c8911ac2b99d8_Wyju3SrkXUDjfkgfD1sdmfZbRUN99w94OjnHX9fl4-Fcq01L-bRiLSBQ0ubN7ndaKZK8U8BxsSyKs-Iiw8KtP-3q63ftyM4NhtW7MTsETWg7g6sqDDQOce6IGWLyeok-W4r6cSpb.png)

**Step 3**

Execute the  `profile-hermes`  command from CLI.

You can use the command  `react-native profile-hermes`  from the React Native CLI to pull the converted Chrome profile to your local machine. Please note that the command only works if the app is run in Development mode since the command uses  `adb pull`  to download the profile from the user's Android device.

Let's dive a little deeper into the steps of your development flow.

First, you want to obtain source maps which will help the profile associate trace events with the application code. While this is optional, it is still highly recommended that source maps are provided to help provide better insights about app performance. You can do that simply by enabling bundleInDebug if the app is running in development mode like so:

In your app's  `android/app/build.gradle`  file, add:

```json
project.ext.react = [
  bundleInDebug: true,
]
```

Be sure to clean the build whenever you make any changes to  `build.gradle`

Clean the build by running:

```bash
cd android && ./gradlew clean
```

Run your app:

```bash
npx react-native run-android
```

Run the command to download the converted profile:

```bash
npx react-native profile-hermes [destinationDir]
```

You can read more about the usage of the command, including the optional arguments it takes, in the documentation  [here](https://github.com/react-native-community/cli/blob/master/docs/commands.md#profile-hermes).

**Step 4**

Open the downloaded profile on Chrome DevTools.

To visualize the downloaded profile in step 3 above in Chrome DevTools, do the following:

-   Open Chrome DevTools.
-   Select the Performance tab.
-   Right-click and choose Load profile...

![Loading a performance profile on Chrome DevTools](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd9619074d36c2d175_VCMe9s8czgULbLL5O7Cy_fXEuPzJWOCFGMh6BcScvnp7ckU_i73idCENh0z-ZdxCra_eTOFSNrTShkYrlSgsWfRf9p-ozU3PccO84s-JRNq-AE_EuRsrYoe2RdjoES09_hDzISZ1.png)

Now you can visualize your app's runtime performance by taking a look at the frequency and duration of each function call. We will suggest some insights we get from the profile in the next part.

## **Usage: With Vanilla JavaScript**

Hermes is a JavaScript engine so it can also be used to profile vanilla JavaScript outside of a React Native application as well.

Hermes can be installed on your system to execute vanilla JavaScript by using the  [jsvu](https://www.npmjs.com/package/jsvu)package on npm. After installing Hermes, we can run our files with Hermes by simply running

```bash
hermes index.js
```

The advantages of profiling still remain, as we can identify efficient implementations of functions. To demonstrate this, we can profile a simple function to calculate the nth Fibonacci number.

Let's take two approaches as an example. We know for a fact that the recursive implementation of calculating the Fibonacci number is slower than the Dynamic Programming implementation.

We can verify that by profiling our Javascript code. Simply run your function with the  `--sample-profiling`  flag. Here's the command to do just that:

`hermes --sample-profiling ./index.js 2> trace-hermes.json`

![](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f877cf68fd6da80ffe89644_Screen%20Shot%202020-10-14%20at%203.32.35%20PM.png)

## **Insights From Profiling Information**

The duration of a function call can be identified from the timestamps of the corresponding start and end events. If this information is successfully captured, the events show up in horizontal bars.

![Profile](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd2452dd1c5ac6eb4e_MfjcXmCNV-arfpTl0nd9AS1PqI-OJsDpTXyYbNZDdxd0AjHYcNCEqyvnXhIpFQT58Wx8rnGiCT_INdX8EYbLa93XhzhO1miDkPC_mmjHR1dQa5HE6HLakum-H_jDEIxiLbbISqIX.png)

As shown in the screenshot, the functions can be seen in horizontal lines, and the horizontal time axis can help us determine how long a function runs. The function calls can also be selected for extra details:

![EventInfo](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd70e3920bdc1c6247_FoTEKIPWHrW2pqDzC4qs46-E9R2WVwksnOnf9KihS6gx_P8VeaOqt5QvIBZ7AdGTOoePA0AsYh-YoQEnopNFprijjwYpgiuFRacnHuYBe7zznIsIjGfA9oKSTwMpPdsvPtR1UB7n.jpeg)

These summaries explicitly contain how long the function runs and we can also identify the line and column numbers by means of which we can also isolate parts of our code base that slows us down.

Source maps add a lot of value here. By default, the line and column numbers are indicated in bundle files. However, when we use source maps, we can map our function calls to the "unbundled" files or user-written source code, improving the debugging experience.

The categories of events help us determine the color of the function rows in the visualization. We broadly have 2 types of categories:

1.  Obtained from source maps - Source maps can be optionally provided to augment the information provided by Hermes. The source maps help us identify better categories for events, hence adding value to our visualization. These categories include two broad categories:

```
   `react-native-internals`
```

```
   `other_node_modules`
```

2.  Obtained from Hermes Samples - These categories are obtained by default and can be mapped to function calls. Categories Javascript and Native are predominantly seen, and in tandem with Source maps, this can help us differentiate from the boilerplate code written in node_modules and the actual code that we write.

As an example from the above profile in the image, we can identify some functions which have a long duration. For instance,  `batchedUpdates$1`  has both high frequency and long duration. It is called 10 times at the zoom level of the image and takes different times on each call. We can hence note this is a crucial function if we wanted to optimize our code for speed. (This function however is internal to  `react-native`, having the category of  `react-native-internals`, hence knowing when and for how long this function is called can be used by React Native core developers to improve React Native performance).

Another example is the  `unstable_runWithPriority`  function which belongs to the category  `other_node_modules`  spanning for ~1 seconds. We notice that the color of this particular bar is different as well. In this case, this is from the  `scheduler`package, which can be understood by reading the summary of the function call in Chrome DevTools. The function calls are color-coded to denote their origin. This can hence help us identify which functions are being invoked from npm packages and how they affect the performance of our application.

We hope that this article has given you useful information on profiling your React Native app using Hermes and visualizing its performance on Chrome DevTools. The new command  `react-native profile-hermes`  currently works only in development mode. However, this can be used in production also, as more details will come soon.

## About G2i

‍**G2i pairs incredible companies with incredibly talented developers**

G2i is the  _only_  marketplace for specialized developers. We focus on finding and vetting JavaScript developers that have deep domain knowledge in one or two areas. When you hire a React Native developer on the G2i platform, they will have passed a test specifically designed to vet their intricate knowledge of the ins and outs of React Native.

We believe in diving deep into one or two pools of knowledge, as opposed scratching the surface of multiple areas of knowledge. With a deep knowledge base developers can hit the ground running immediately. Our litmus test is: "Can this developer make an impact in your codebase in the first week?"

Our vetting is extremely deep. Each developer goes through cultural vetting, language assessment, a code challenge, and a technical interview that are specific to the tech stack the developer wants to work with. For example, if a developer that you hire wants to work with React, they will have already passed our React-specific coding challenge.

We believe in complete transparency in the hiring process. You'll receive technical profiles of each developer that include our detailed assessment of their code challenge and technical interview. We also provide the code they wrote and access to a recording of their technical interview.  [Learn more](https://www.g2i.co/)  about how G2i can pair you with the perfect developer for your project and budget.