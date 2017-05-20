# First release of a stable version of Aranea.HttpMessageHandler

Yesterday I published a first stable release of Aranea.HttpMessagehandler. It is an implementation of System.Net.Http.HttpMessageHandler that translates an HttpRequestMessage into an ASP.NET Core compatible HttpContext, calls the supplied RequestDelegate and translates the result to an HttpResponseMessage. This allows you to call an ASP.NET Core application (RequestDelegate) / Middleware using an HttpClient without actually hitting the network stack. Useful for testing and embedded scenarios.

It is port of Damian Hickey's [OwinHttpMessageHandler](https://github.com/damianh/OwinHttpMessageHandler) so that the concept can be used for ASP .NET core as well.

Source can be found on [GitHub](https://github.com/MCGPPeters/Aranea.HttpMessageHandler).

Install via NuGet: [![NuGet Latest Stable https://www.nuget.org/packages/Aranea.HttpMessageHandler](https://img.shields.io/nuget/v/Aranea.HttpMessageHandler.svg)](https://www.nuget.org/packages/Aranea.HttpMessageHandler)

