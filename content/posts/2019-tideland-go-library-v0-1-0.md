+++
date = "2019-04-18T20:35:00+01:00"
draft = false
title = "Tideland Go Library v0.1.0"
tags = ["golang", "tideland", "library", "release"]
categories = ["tideland"]
+++

It costed some time, but now I'm happy to release **v0.1.0** of the **Tideland Go Library**. Some of you may wonder, because I already had a Go library and once started to break it into individual repositories. But then different discussions happened.

1. The introduction of Go modules.
2. The discussions about mono-repos or multi-repos.
3. Microsoft was buying GitHub with the insight about the dependency of one provider, regardless who it is.

So after some thinking I decided to create a *mono-repo* as module and using an own domain. This also would be a good opportunity to fix some old design decisions I won't repeat today, i.e. the overly usage of exported interfaces with only one private implementation.

The repository itself including issue handling and release planning is still at [GitHub](https://github.com/tideland/go), it's working fine. But the used domain for my software  is **tideland.dev**. So the `asserts` package now has to be imported via

```go
import tideland.dev/go/audit/asserts
```

To achieve this I'm using the remote import paths (see documentation of [go command](https://golang.org/cmd/go/#hdr-Remote_import_paths)). Here I told the *nginx* in my according Docker container to send a document containing the according meta tag as reply if the request contains the argument `go-get`.

```nginx
location ~ "(/[^/]+)(/.*)?" {
    if ($arg_go-get = "1") {
        echo '<html><head><meta name="go-import" content="tideland.dev$1 git https://github.com/tideland$1.git"/></head></html>';
    }
    root /tideland;
}
```

The root directory also contains an `index.html` redirecting to the category [Tideland](https://themue.dev/categories/tideland/) here on this site. Additionally I'm mirroring the repository in this container but don't provide access to it right now. For cloning you still have to go to GitHub.

Another part of it has been to add an import comment due to different domain (see documentation of [go command](https://golang.org/cmd/go/#hdr-Import_path_checking) again).

```go
package asserts // import "tideland.dev/go/audit/asserts
```

When modules are used everywhere they will become obsolete. The Tideland Go Library now already uses them and they work like a charm.

The provided packages themselves best can be seen in the [README](https://github.com/tideland/go). They are grouped in

- testing and testing support,
- database clients,
- data structures and algorithms,
- networking,
- text data processing,
- concurrency, and
- taking a deeper look what's happening in your application.

First planning issues for the v0.2.0 already get collected. Biggest block will be the migration of the *cells*. Feel free to add issues for feature request and bug—here not too much—to the repository on GitHub.
