---
layout: post
title: Injecting dependecies into Web API Action Filters

excerpt: "When I was working with ASP.NET Web API I needed to inject dependencies to ActionFilters using Ninject."
---

When I was working with ASP.NET Web API I needed to inject dependencies to ActionFilters using Ninject. This is solved by [Ninject.Web.WebApi][1], but I had some stability issues with this library. When I started to dig a little deeper I found that in fact I'm using only one class from this library. Which is [DefaultFilterProvider][2] this class steps into play before filter methods are executed and injects dependencies into it.

    public IEnumerable<FilterInfo> GetFilters(HttpConfiguration configuration,
                                            HttpActionDescriptor actionDescriptor)
    {
        var filters = this.filterProviders.SelectMany(fp => 
                fp.GetFilters(configuration, actionDescriptor)).ToList();
        
        foreach (var filter in filters)
        {
            this.kernel.Inject(filter.Instance);
        }
        
        return filters;
    }

And how do we tell the WEB API that we want to use this provider? First we need to register our provider to GlobalConfiguration services and then remove the default one.

    var providers = GlobalConfiguration.Configuration.Services.GetFilterProviders().ToList();
    GlobalConfiguration.Configuration.Services.Add(typeof (IFilterProvider),
                                                   new DefaultFilterProvider(ninjectKernel, providers));
                
    var defaultprovider = providers.First(i => i is ActionDescriptorFilterProvider);
    GlobalConfiguration.Configuration.Services.Remove(typeof(IFilterProvider), defaultprovider);

Example is available on [GitHub][3].


  [1]: https://github.com/azzlack/Ninject.Web.WebApi "Link to GitHub project"
  [2]: https://github.com/azzlack/Ninject.Web.WebApi/blob/master/src/Ninject.Web.WebApi/Filter/DefaultFilterProvider.cs
  [3]: https://github.com/stlk/WebApiDependencyInjectionWithNinject