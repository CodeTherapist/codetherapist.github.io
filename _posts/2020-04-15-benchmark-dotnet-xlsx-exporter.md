---
layout: post
title: "How to implement a BenchmarkDotNet XLSX exporter"
date: 2020-04-15
---

<p class="intro">
    <span class="dropcap">L</span>et's implement a BenchmarkDotNet exporter, that writes a xlsx file with the benchmark results.
</p>
<br/>

_The [BenchmarkDotNetXlsxExporter](https://github.com/CodeTherapist/BenchmarkDotNetXlsxExporter){:target="_blank"} is open source and published as [nuget package](https://www.nuget.org/packages/BenchmarkDotNetXlsxExporter/){:target="_blank"}._

### Requirements & Information gathering

Usually before I start writing any code, there should be an added value of the purposed solution (feature).
So, what is the main use case/feature/user story or however you name it?
The purpose of the library is simple:

* A BenchmarkDotNet exporter that writes the results into a xlsx file

Why is that useful - isn't the built-in csv exporter not enough?
Yes and no. The xlsx format has some benefits:

* It can hold many spreadsheets and store more information in a convenient way, instead of multiple csv files
* Compatible with Open Office and Office 365
* Further validation and virtualization possibilities (e.g. charts)

When the use case is identified and considered as real value, it's common to validate if and how it is feasible to be implemented.
In this case, the interface to [BenchmarkDotNet](https://benchmarkdotnet.org/){:target="_blank"} is given.
[All exporters in BenchmarkDotNet](https://benchmarkdotnet.org/api/BenchmarkDotNet.Exporters.IExporter.html){:target="_blank"} implementing the following interface:

{% highlight c# %}
public interface IExporter
{
    string Name { get; }

    void ExportToLog(Summary summary, ILogger logger);

    IEnumerable<string> ExportToFiles(Summary summary, ILogger consoleLogger);
}
{% endhighlight %}

As you can see, it isn't complicated to implement a valid exporter that integrates seemlessly into the BenchmarkDotNet ecosystem.
The second part is writing the data as xlsx file - for this, I use the [official OpenXML nuget package](https://www.nuget.org/packages/DocumentFormat.OpenXml/){:target="_blank"}.

Keep in mind that, this "process" is most of the time a lot harder and takes more time as it is for this small library.
But as usual, I connect simplicity and usfulness when writing blog posts.

### Basic project setup

Let's start Visual Studio 2019 and create a class library project targeting `netstandard2.0`.
Additionaly, a test project for unit tests seems to be a good idea.
Further, I add a console app as show case how the exporter could be used.

### Let's write some unit tests!

Why on earth do I start with writing a unit test, there isn't any code to test?!
Yes, it's right. There isn't any code. Probably you heard about [TDD (Test Driven Development)](https://en.wikipedia.org/wiki/Test-driven_development){:target="_blank"}.
In a nutshell, it's writing tests for your (core) requirements and implementing the code until the test is successful (green).
I'm using this concept when I see it fits for the project. 
Beside that, I'm using unit tests as playground to explore third party APIs and find out how my solution could work.

The core requirement of this library is to export the BenchmarkDotNet summary as xlsx file.
That is why, it's a good idea to start having this tested and working: 

{% highlight c# %}
[Fact]
public void ExportToFilesTest()
{
    string file = null;
    try
    {
        var summary = TestBenchmarkSummaries.GetSummary();
        var xlsxReporter = new XlsxExporter();
        var files = xlsxReporter.ExportToFiles(summary, NullLogger.Instance);

        Assert.True(files.Any());
        file = files.First();
        Assert.True(File.Exists(file));
        Assert.True(File.GetLastWriteTime(file) > DateTime.Now.AddSeconds(-10));
    }
    finally
    {
        if (!(file is null))
            File.Delete(file);
    }
}
{% endhighlight %}

This test is more like an integration test - it runs a real benchmark and uses the output of it.
_Note: Unfortunaelty I did not find a good way of "mock" the `Summary` class._

The method of the interface `IExporter.ExportToFiles(Summary summary, ILogger consoleLogger)` requires a physical file on the disk. 
For unit testing purpose, this isn't optimal - imagine, there is for example an access authorization issue when the file should be written. 
The test would fail, even there is nothing wrong with code itself (false-positive) and this should be avoided - the test should reveal issues with the code under test, not with any of it's dependencies. Additionally, the test execution time could be negatively influenced when the file system is used.

Further the file system brings another issue: was the file really written or is it two days old?
Probably you noticed the last assertion with the last write time of the file and the `File.Delete(file)` within the `finally` block. 
This is to ensure, the file was really written. I know, this is still not 100% reliable but fair enough. Let me know how you solve that kind of issue.

It would make sense to add a method to the `XlsxExporter` class, that uses an output `stream` as input and is called internally by the method `IEnumerable<string> ExportToFiles(Summary summary, ILogger consoleLogger)`:

{% highlight c# %}
public void CreateSpreadsheetWorkbook(Stream stream, Summary summary, ILogger consoleLogger) => { }
{% endhighlight %}

This solves at least the file system dependency:

{% highlight c# %}
[Fact]
public void CreateSpreadsheetWorkbookTest()
{
    using (var filePath = TestHelper.GetTemporaryFilePath("CreateSpreadsheetWorkbookTest.xlsx"))
    {
        var summary = BenchmarkRunnerForTests.GetSummary();

        var xlsxReporter = new XlsxExporter();
        xlsxReporter.CreateSpreadsheetWorkbook(filePath, summary, NullLogger.Instance);

        Assert.True(File.Exists(filePath));
    }
}
{% endhighlight %}

Note: This doesn't replace the previous test. The interface method `IExporter.ExportToFiles(...)` should be tested at least to make sure there isn't an exception thrown when calling it with valid parameters.

### Details of the implementation

Let's dive into some details... 
When you ever used the [Open XML SDK](https://docs.microsoft.com/en-us/office/open-xml/open-xml-sdk){:target="_blank"}, you probably noticed, that it is low level abstraction over the [OpenXML file format](https://en.wikipedia.org/wiki/Office_Open_XML){:target="_blank"}. More specifically, the [SpreadsheetML](https://docs.microsoft.com/en-us/office/open-xml/structure-of-a-spreadsheetml-document){:target="_blank"} is relevant for this implementation. That is why, I decided to use [facades](https://en.wikipedia.org/wiki/Facade_pattern){:target="_blank"} to simplify the complexity within the exporter.

In general, I follow the [SOLID](https://en.wikipedia.org/wiki/SOLID){:target="_blank"} principles as much as I can.
I created an interface `IXlsxExporterHandler` to handle the data flow into the xlsx file.
The exporter has a list of those and executes them (see code below). 
For instance, this is the `Open-Close principle` - you can always add/remove a handler of type `IXlsxExporterHandler` without changing the core of the `XlsxExporter`. This encourages also the `Single-responsibility principle`, one handler has exactly one purpose. 
Further, when you can use a class easily outside of it's normal scope, this indicates loose/low coupling.
Most of the time, loose/low coupling is a good thing to have because it increases the overall maintainability and testability of a library/product.
The interface `IXlsxExporterHandler` also meant to be as an extensibility point for other developers to extend the xlsx output.

{% highlight c# %}
public void CreateSpreadsheetWorkbook(Stream stream, Summary summary, ILogger consoleLogger)
{
    // parameter checks omitted.

    using (var spreadsheetDocument = SpreadsheetDocument.Create(stream, SpreadsheetDocumentType.Workbook))
    {
        var spreadsheet = new XlsxSpreadsheetDocument(spreadsheetDocument);
        spreadsheet.InitializeWorkbook();

        // "handlers" is our list to be executed:
        var handlers = _benchmarkXlsxHandlers.Any() ? _benchmarkXlsxHandlers : MinimalXlsxHandlers;
        foreach (var handler in handlers)
        {
            try
            {
                handler.Handle(spreadsheet, summary);
            }
            catch (Exception ex)
            {
                consoleLogger.WriteLineError($"Cannot execute {handler.GetType()}: {ex.ToString()}.");
            }
        }
        spreadsheet.Save();
    }
}
{% endhighlight %}

### Summary

Hopefully, you got some (mini) insights of how I develop. It was quite fun to work on this small project.

As always, feel free to do pull requests for any kind of improvements, ask questions or just star the repository when you find it useful!

