<img src="https://raw.github.com/strumenta/SmartReader/master/logo.png" width="64">

**SmartReader** is a [.NET Standard 1.3](https://github.com/dotnet/standard/blob/master/docs/versions/netstandard1.3.md) library to extract the main content of a web page, based on a port of the [Readability](https://github.com/mozilla/readability) library by Mozilla, which in turn is based on the famous original Readability library.

## Installation

You can do it the standard way, by using the [NuGet](https://www.nuget.org/packages/SmartReader/) package.

```
PM> Install-Package SmartReader
```

## Why You May Want To Use It

 There are already other similar good projects, but they don't support .NET Core and they are based on old version of Readability. The original library is already quite stable, but there are always improvement to be made. So by relying on a original library maintained by such a competent organization we can piggyback on their hard work and user base.

 There are also some improvements: it returns an author and publication date, the language of the article, the featured image, a list of images and an indication of the time needed to read it.

 Feel free to suggest new features. 

## Usage

There are mainly two ways to use the library. The first is by creating a new `Reader` object, with the URI as the argument, and then calling the `GetArticle` method to obtain the extracted `Article`. The second one is by using one of the static methods `ParseArticle` of `Reader` directly, to return an `Article`. Both ways are available also through an async method, called respectively `GetArticleAsync` and `ParseArticleAsync`.
The advantage of using an object, instead of the static method, is that it gives you the chance to set some options.

There is also the option to parse directly a String or Stream that you have obtained by some other way. This is available either with `ParseArticle` methods or by using the proper `Reader` constructor. In either case, you also need to give the original URI. It will not re-download the text, but it need the URI to make some checks and modifications on the links present on the page. If you cannot provide the original uri, you can use a fake one, like `https:\\localhost`.

If the extraction fails, the returned `Article` object will have the field `IsReadable` set to `false`.

The content of the article is unstyled, but it is wrapped in a `div` with the id `readability-content` that you can style yourself.

The library tries to detect the correct encoding of the text, if the correct tags are present in the text.

On the `Article` object you can call `GetImagesAsync` to obtain a Task for a list of `Image` objects, representing the images found in the extracted article. The method is async because it makes HEAD Requests, to obtain the size of the images and only returns the ones that are bigger then the specified size. The size by default is 75KB.
This is to exclude things such as images used in the UI.

## Examples

Using the `GetArticle` method.

```csharp
SmartReader.Reader sr = new SmartReader.Reader("https://arstechnica.co.uk/information-technology/2017/02/humans-must-become-cyborgs-to-survive-says-elon-musk/");

sr.Debug = true;
sr.Logger = new StringWriter();

SmartReader.Article article = sr.GetArticle();
var images = article.GetImagesAsync();

if(article.IsReadable)
{
	// do something with it	
}
```

Using the `ParseArticle` static method.

```csharp

SmartReader.Article article = SmartReader.Reader.ParseArticle("https://arstechnica.co.uk/information-technology/2017/02/humans-must-become-cyborgs-to-survive-says-elon-musk/");

if(article.IsReadable)
{
	// do something with it
}
```
## Options

- `int` **MaxElemsToParse**<br>Max number of nodes supported by this parser. <br> *Default: 0 (no limit)*
- `int` **NTopCandidates** <br>The number of top candidates to consider when analyzing how tight the competition is among candidates. <br>*Default: 5*
- `bool` **Debug** <br>Set the Debug option. If set to true the library writes the data on Logger.<br>*Default: false*
- `TextWriter` **Logger** <br> Where the debug data is going to be written. <br> *Default: null*
- `bool` **ContinueIfNotReadable** <br> The library tries to determine if it will find an article before actually trying to do it. This option decides whether to continue if the library heuristics fails. This value is ignored if Debug is set to true <br> *Default: true*
- `int` **WordThreshold** <br>The minimum number of words an article must have in order to return a result. <br>*Default: 500*

## Article Model

- `Uri` **Uri**<br>Original Uri
- `String` **Title**<br>Title
- `String` **Byline**<br>Byline of the article, usually containing author and publication date
- `String` **Dir**<br>Direction of the text
- `String` **FeaturedImage**<br>The main image of the article
- `String` **Content**<br>Html content of the article
- `String` **TextContent**<br>The pure text of the article
- `String` **Excerpt**<br>A summary of the article, based on metadata or first paragraph
- `String` **Language**<br>Language string (es. 'en-US')
- `int` **Length**<br>Length of the text of the article
- `TimeSpan` **TimeToRead**<br>Average time needed to read the article
- `DateTime?` **PublicationDate**<br>Date of publication of the article
- `bool` **IsReadable**<br>Indicate whether we successfully find an article

It's important to be aware that the fields **Byline**, **Author** and **PublicationDate** are found independently of each other. So there might be some inconsistencies and unexpected data. For instance, **Byline** may be a string in the form "@Date by @Author" or "@Author, @Date" or any other combination used by the publication. 

The **TimeToRead** calculation is based on the research found in [Standardized Assessment of Reading Performance: The New International Reading Speed Texts IReST](http://iovs.arvojournals.org/article.aspx?articleid=2166061). It should be accurate if the article is written in one of the languages in the research, but it is just an educated guess for the others languages.

The **FeaturedImage** property holds the image indicated by the Open Graph or Twitter meta tags. If neither of these is present, and you called the `GetImagesAsync` method, it will be set with the first image found. 

## Demo & Console Projects

The demo project is a simple ASP.NET Core webpage that allows you to input an address and see the results of the library.

The console project is a Console program that allows you to see the results of the library on a random test page.

## Creating The Nuget Package

In case you want to build the Nuget package yourself you cannot use the standard `nuget pack` because of a [bug related to .NET Core](https://github.com/NuGet/Home/issues/4491). Instaed use the dotnet pack command.

```
dotnet pack --configuration Release --output "..\nupkgs"
```

The command must be issued inside the `src/SmartReader` folder, otherwise it will generate nuget packages for all projects.

## License

The project uses the **Apache License**.

## Contributors

- [Unosviluppatore](https://github.com/unosviluppatore)
- [DanRigby](https://github.com/DanRigby)
- [Yasindn](https://github.com/yasindn)
- [jamie-lord](https://github.com/jamie-lord)

Thanks to all the people involved.