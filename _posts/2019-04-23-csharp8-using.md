---
layout: post
title:  "C# 8: Using statement"
date:   2019-04-23
---

<p class="intro">
    <span class="dropcap">S</span>implify your using statements.
</p>

<br/>

### C# 8.0 Series

Want to read my other posts about C# 8?

* <a href="https://codetherapist.github.io/blog/csharp8-nullable-ref-types/" target="_blank">C# 8: nullable reference types</a>.
* <a href="https://codetherapist.github.io/blog/csharp8-indexes-ranges/" target="_blank">C# 8: Indexes and Ranges</a>.
* C# 8: Using statement revisited!

### Prerequisites & Setup

You will need [Visual Studio 2019](https://visualstudio.microsoft.com/vs/preview/) and [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) to try out the new using statement syntax. We need to modify the .csproj file to enable C# 8.0 as well:

{% highlight xml %}

<LangVersion>8.0</LangVersion>

{% endhighlight %}

### A small example

Did you liked my _Car_ class from the previous C# 8 posts? No? That's okay, I will anyway use it again ;).
This time the whole code is little more advanced because we will encrypt and decrypt our car.

{% highlight c# %}
class Program
    {
        static void Main(string[] args)
        {
            var car = new Car
            {
                Name = "My awesome car!"
            };

            var key = "MySecret";
            var keyIV = "MyIVSecret";
            byte[] rgbkey;
            byte[] rgbIV;

            using (var pbkdf = new Rfc2898DeriveBytes(key, 10))
            rgbkey = pbkdf.GetBytes(32);

            using (var pbkdf = new Rfc2898DeriveBytes(keyIV, 10))
            rgbIV = pbkdf.GetBytes(16);

            Car decryptedCar = null;
            using (var outStream = new MemoryStream())
            {
                Encrypt(car, outStream, rgbkey, rgbIV);

                // rewind the stream to be readable.
                outStream.Position = 0;

                decryptedCar = Decrypt<Car>(outStream, rgbkey, rgbIV);
            }

            // ...
        }
    }
{% endhighlight %}

I hope you aren't overwhelmed by this - don't worry I will explain it as good as I can.
On the first few lines we create an instance of a _Car_ and generate a secret key and initialization vector for the encryption and decryption.

<p class="alert alert-danger">
    <b>Please be aware: Never store a secret (passwords, useranmes, keys, ...) in your code! Seriously, it isn't secure in any way!</b>
</p>

After that we encrypt the car and put the result into a memory stream named _outStream_.
Then we need to rewind the stream to position zero to ensure we are able to read (decrypt) it.
The decrypt function is returning a new instance of the car that must hold the same information, as the first instance.
Otherwise we messed up our encrypt or decrypt function (btw. that would be a good candidate for a Unit-Test).

Everyone who reads attentively spotted already that my _using_ statement looks incomplete for prior C# versions (<=7.3).
Are you familiar with the concept of stream-chaining? When not, then I show you something new combined with the new _using_ syntax.
Let's see the _Encrypt_ method used above:

{% highlight csharp %}
public static void Encrypt<T>(T obj, Stream outStream, byte[] rgbKey, byte[] rgbIV)
{
    using var rijndael = Rijndael.Create();
    using var cryptoTransform = rijndael.CreateEncryptor(rgbKey, rgbIV);
    using var cryptoStream = new CryptoStream(outStream, cryptoTransform, CryptoStreamMode.Write, leaveOpen: true);
    using var deflateStream = new DeflateStream(cryptoStream, CompressionLevel.Fastest);
    var binaryFormatter = new BinaryFormatter();
    binaryFormatter.Serialize(deflateStream, obj);
    // disposing deflateStream
    // disposing cryptoStream
    // disposing cryptoTransform
    // disposing rijndael
}
{% endhighlight %}

Here, we take the _outStream_ parameter and pass it to the _CryptoStream_ to have everything written to the stream encrypted.
The _cryptoStream_ is passed to the _DeflateStream_ to get the stream compressed as well.
Finally we use the _BinaryFormatter_ to serialize what ever instance we pass in as _obj_.

Beside the encryption, stream-chaining and serialization - important is that almost everything is disposable here and ultimately shows how much cleaner the new capability of the _using_ statement looks like. But still maintaining disposing all instances as soon they left the scope.
The comments are reflecting the order of how they get disposed.

This new capability doesn't change the framework or the runtime - it is a pure language feature.
Roslyn emits the rest of the _using_ statement for us!
IL-Spy reveals this:

![il-spy-using-statement](/assets/img/csharp8-using-statement/il-spy-using-statement.png)

For completeness the _Decrypt_ method:

{% highlight csharp %}
public static T Decrypt<T>(Stream inStream, byte[] rgbKey, byte[] rgbIV)
{
    using var rijndael = Rijndael.Create();
    using var cryptoTransform = rijndael.CreateDecryptor(rgbKey, rgbIV);
    using var cryptoStream = new CryptoStream(inStream, cryptoTransform, CryptoStreamMode.Read);
    using var deflateStream = new DeflateStream(cryptoStream, CompressionMode.Decompress);
    var binaryFormatter = new BinaryFormatter();
    return (T)binaryFormatter.Deserialize(deflateStream);
}
{% endhighlight %}

### Closing words

This feature is small, but has enough impact to be worth writing and thinking about it.
I love it and I'm sure you will use it too.
It removes some tedious steps when we have local disposable resource as shown above especially when we forget the using statement.
Writing less code usually increases also readability and this is important.
Code is less written than read and therefore it is critical for our colleagues and ourself.