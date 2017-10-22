# AspNetCore.Identity.MongoDbCore
A MongoDb UserStore and RoleStore adapter for Microsoft.AspNetCore.Identity 2.0.
Allows you to use MongoDb instead of SQL server with Microsoft.AspNetCore.Identity 2.0.

Covered by 737 integration tests and unit tests from the modified [Microsoft.AspNetCore.Identity.EntityFrameworkCore.Test](https://github.com/aspnet/Identity/tree/b865d5878623077eeb715e600d75fa9c24dbb5a1/test/Microsoft.AspNetCore.Identity.EntityFrameworkCore.Test) test suite.

# Usage examples

Your user and role entities must inherit from MongoIdentityUser<Guid> and MongoIdentityRole<TKey> in a way similar to the IdentityUser<TKey> and the IdentityRole<TKey> in Microsoft.AspNetCore.Identity.
Here is an example:

```csharp

	public class ApplicationUser : MongoIdentityUser<Guid>
	{
		public ApplicationUser() : base()
		{
		}

		public ApplicationUser(string userName, string email) : base(userName, email)
		{
		}
	}
	
	public class ApplicationRole : MongoIdentityRole<Guid>
	{
		public ApplicationRole() : base()
		{
		}

		public ApplicationRole(string roleName) : base(roleName)
		{
		}
	}	
```
The `Id` field is automatically set at instanciation, this also applies to users inheriting from `MongoIdentityUser<int>`, where a random integer is assigned to the `Id`. It is however not advised to rely on such random mechanism to set the primary key of your document. Using documents inheriting from `MongoIdentityRole<Guid>` is recommended.

The configuration is done by populating a `MongoDbIdentityConfiguration` object, which can have an `IdentityOptionsAction` property set to an action you want to perform against the `IdentityOptions` (`Action<IdentityOptions>`).

The `MongoDbSettings` object is used to set MongoDb Settings using the `ConnectionString` and the `DatabaseName` properties.

The MongoDb connection is managed using the [mongodb-generic-repository](https://github.com/alexandre-spieser/mongodb-generic-repository), where a repository inheriting `IBaseMongoRepository` is registered as a singleton. Look at the [ServiceCollectionExtension.cs](https://github.com/alexandre-spieser/AspNetCore.Identity.MongoDbCore/blob/master/src/Extensions/ServiceCollectionExtension.cs) file for more details.

```csharp
        /// <summary>
        /// This method gets called by the runtime. Use this method to add services to the container.
        /// </summary>
        /// <param name="services"></param>
        public void ConfigureServices(IServiceCollection services)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(_hostingEnvironment.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                .AddJsonFile($"appsettings.{_hostingEnvironment.EnvironmentName}.json", optional: true);


            Configuration = builder.Build();

            services.AddOptions();
			
            // add a global config object
            services.AddSingleton(Configuration);
	    // get the MongoDb settings from the Configuration object.
            var settings = Configuration.GetSection(nameof(MongoDbSettings)).Get<MongoDbSettings>();
            var mongoDbIdentityConfiguration = new MongoDbIdentityConfiguration
            {
                MongoDbSettings = settings,
                IdentityOptionsAction = options =>
                {
                    options.Password.RequireDigit = false;
                    options.Password.RequiredLength = 8;
                    options.Password.RequireNonAlphanumeric = false;
                    options.Password.RequireUppercase = false;
                    options.Password.RequireLowercase = false;

                    // Lockout settings
                    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(30);
                    options.Lockout.MaxFailedAccessAttempts = 10;

                    // ApplicationUser settings
                    options.User.RequireUniqueEmail = true;
                    options.User.AllowedUserNameCharacters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789@.-_";
                }
            };
            services.ConfigureMongoDbIdentity<ApplicationUser, ApplicationRole, Guid>(mongoDbIdentityConfiguration);
        }
```

## Author
**Alexandre Spieser**

## Donations
Feeling like my work is worth a coffee? 
Donations are welcome and will go towards further development of this project as well as other MongoDb related projects. Use the wallet address below to donate.
BTC Donations: 1FDMWqSK8SHXDGKKp7gyZc4rknynWJ7qbj

*Thank you for your support and generosity!*

## License
AspNetCore.Identity.MongoDbCore is under MIT license - http://www.opensource.org/licenses/mit-license.php

The MIT License (MIT)

Copyright (c) 2016-2017 Alexandre Spieser

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

## Copyright
Copyright © 2017