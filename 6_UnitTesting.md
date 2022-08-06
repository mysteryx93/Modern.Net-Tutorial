# Tutorial: Build Modern Cross-Platform Apps with .NET

[1. Introduction](README.md)  
[2. Avalonia UI](2_Avalonia.md)  
[3. Dependency Injection](3_DependencyInjection.md)  
[4. MVVM Design](4_MVVM.md)  
[5. Dialogs and Tools](5_DialogsTools.md)  
[6. Unit Testing](6_UnitTesting.md)  
[7. Reactive](7_Reactive.md)  
[8. Deployment](8_Deployment.md)

## 6. Unit Testing

Just like MVVM ravioli code is longer to write but requires less changes and is more expandable, unit testing can double your programming time but cut down debugging by 10x. If you've ever tried to estimate your time on a project, whatever time you need has to be multiplied by 4-10x because of testing and debugging. While Unit Testing can represent 50% of your coding time, it can drastically reduce debugging time while greatly increasing the code quality. You would be surprised: you think your code is great, but unit tests rarely pass on the first try!

On a personal note, it has been difficult to adapt my mind to the unit test mindset. Some say that it takes about a year to get used to it. It's all about mindset. If you structured your code correctly from the start, then it makes things a lot easier. Whether or not you want to create unit tests now, do read this section to understand how to structure your code to allow it.

Here are situations I faced where unit tests really shine:

- I created a wrapper to a complex online API ([OntraportApi.Net](https://github.com/mysteryx93/OntraportApi.NET)). First, their services can change and introduce bugs or different behaviors at any time, it allows me to detect problems. Morever, I refactored the code. It allows me to know that the public API maintains the same behaviors! Tests rarely work on first try; there were a lot of broken details to fix. Imagine trying to fix the API manually after refactoring? It would leave tons of functions untested and broken.

- I created a recurring payment processing system. There are like 100 different scenarios that can occur! How the heck would you test that manually? There is a single method that gets called by a webhook when a payment comes in; you need to test that single method in a hundred different ways. I can't imagine doing that manually.

Some sitations where unit testing is harder: user controls and extensions methods. I do not recommend starting to learn with those. I recommend starting by testing some back-end services.

### How to Code with Unit Tests

