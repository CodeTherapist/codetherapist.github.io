---
layout: post
title:  "dotnet http-security-check"
date:   2019-01-07
---

<p class="intro">
    <span class="dropcap">T</span>his global dotnet tool helps to secure your web application.
</p>

As everyone should know: **security is important and critical**  - but not easily done right.
The attack surface _especially_ for public websites is fairly large and keeping everything secure is a challange.
Using security headers and TLS (HTTPS) is a neat possibility to reduce this attack surface effectively.

The global tool <a href="https://github.com/CodeTherapist/DotnetHttpSecurityCheck" target="_blank">DotnetHttpSecurityCheck</a> implements different checks to ensure best practice and suggests improvements. They are splitted into two categories **Header** and **Request**.

* A **Header** check examines the value of a response header field.
* A **Request** check examines any other security related aspect (e.g. valid certificate).

Hopefully, by providing this tool, it helps everyone to assess and reinforce security.

### Installation

Download and install the <a href="https://www.microsoft.com/net/download" target="_blank">.NET Core 2.2 SDK</a> or newer. Once installed, run the following command:

{% highlight cmd %}
dotnet tool install DotnetHttpSecurityCheck -g
{% endhighlight %}

### Execute a scan

After installation, you can use the tool directly from the CLI (command line interface):

{% highlight cmd %}
dotnet-http-security-check https://www.google.ch
{% endhighlight %}

### Analyzing the results

Each check returns a result consisting of:

* Actual value
* Rating (see below)
* Suggestion

<figure>
	<img src="/assets/img/dotnet-security-check-result-explained.jpg" alt="dotnet-security-check-result-explained"> 
	<figcaption>Fig1. - Result explained</figcaption>
</figure>

#### Best

Everything is fine - the currently best known value is set.

<figure>
	<img src="/assets/img/dotnet-security-check-result-best.jpg" alt="dotnet-security-check-result-best"> 
	<figcaption>Fig2. - Example for best result</figcaption>
</figure>

#### Good

The configuration is basically acceptable - but you could improve it accordingly to the suggestion.

<figure>
	<img src="/assets/img/dotnet-security-check-result-good.jpg" alt="dotnet-security-check-result-good"> 
	<figcaption>Fig3. - Example for good result</figcaption>
</figure>

#### Bad

Indicates you should fix the value accordingly to the suggestion - otherwise there is a security risk (e.g. unsecure connection, cross site scripting, ...).

<figure>
	<img src="/assets/img/dotnet-security-check-result-bad.jpg" alt="dotnet-security-check-result-bad"> 
	<figcaption>Fig4. - Example for bad result</figcaption>
</figure>

#### Skipped

This means the check is not applicable for the current request.
For example the 'Strict-Transport-Secuirty' header is only recognized when sent over an HTTPS connection.

<figure>
	<img src="/assets/img/dotnet-security-check-result-skipped.jpg" alt="dotnet-security-check-result-skipped"> 
	<figcaption>Fig5. - Example for bad skipped</figcaption>
</figure>