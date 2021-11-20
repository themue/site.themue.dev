+++
date = "2021-11-19T12:00:00+01:00"
draft = false
title = "RESTful APIs with Go"
+++

## Table of Contents

1. Introduction
	* [Who am I](introduction/whoami.md)
	* [Experience with Go](introduction/experience.md)
	* [Why Go](introduction/whygo.md)
2. Simple APIs with `net/http`
	* [The `net/http` package](nethttp/nethttp.md)
	* [Requests](nethttp/requests.md)
	* [Response](nethttp/response.md)
	* [The building blocks together](nethttp/buildingblocks.md)
3. HTTP methods and the data
	* [Handling HTTP methods](methods/methods.md)
	* [RESTful APIs and CRUD](methods/crud.md)
	* [URI handling with the httpx package](methods/uri.md)
	* [Data and `encoding/json`](methods/json.md)
	* [Body handling with the httpx package](methods/body.md)
	* [The building blocks together](methods/buildingblocks.md)
4. Multiplexing
	* [Distribute paths to handlers](multiplexing/multiplexing.md)
	* [Nesting of handlers](multiplexing/nesting.md)
	* [The building blocks together](multiplexing/buildingblocks.md)
5. Other useful wrappers
	* [Logging](wrappers/logging.md)
	* [JWT for authentication and authorization](wrappers/jwt.md)s
7. Summary
