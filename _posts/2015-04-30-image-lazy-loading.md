---
layout: post
title: Images - Lazy Loading
date: 2015-04-30
header_desc: Images - Lazy Loading
---
<p><span class="step">1</span> Create an attribute filter <code>CompressAttribute</code> and override <code>OnActionExecuting()</code> method.</p>

{% highlight csharp linenos %}

namespace Website.AttributeFilters
{
    public class CompressAttribute : ActionFilterAttribute
    {
        public override void OnActionExecuting(ActionExecutingContext filterContext)
        {
            // Skip if child action is called, as Response.filter is null for child action.
            if (filterContext.IsChildAction) return;

            HttpRequestBase request = filterContext.HttpContext.Request;

            string acceptEncoding = request.Headers["Accept-Encoding"];

            if (string.IsNullOrEmpty(acceptEncoding)) return;

            acceptEncoding = acceptEncoding.ToUpperInvariant();

            HttpResponseBase response = filterContext.HttpContext.Response;

            if (acceptEncoding.Contains("GZIP"))
            {
                response.AppendHeader("Content-encoding", "gzip");
                response.Filter = new GZipStream(response.Filter, CompressionMode.Compress);
            }
            else if (acceptEncoding.Contains("DEFLATE"))
            {
                response.AppendHeader("Content-encoding", "deflate");
                response.Filter = new DeflateStream(response.Filter, CompressionMode.Compress);
            }
        }
    }
}

{% endhighlight %}

<p><span class="step">2</span> Register the newly created custom attribute inside <code>FilterConfig.cs</code>.</p>

{% highlight csharp linenos %}

namespace Website
{
    public class FilterConfig
    {
        public static void RegisterGlobalFilters(GlobalFilterCollection filters)
        {
            filters.Add(new AttributeFilters.CompressAttribute());
        }
    }
}

{% endhighlight %}