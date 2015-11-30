# Module 2 - ASP.NET Web API Security Architecture #

## The Security Pipeline ##

- ASP.NET Web API v1 had dependency on IIS for authentication
	- Windows Authentication
	- Basic Authentication
- The New Pipeline in Web API v2
	- OWIN for Hosting (hosting adapter)
	- All of the new security investments happen in the OWIN layer
		- Social or token-based authentication