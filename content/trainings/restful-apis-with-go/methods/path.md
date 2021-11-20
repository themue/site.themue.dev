+++
date = "2021-11-19T12:00:00+01:00"
draft = false
title = "RESTful APIs with Go - Paths in RESTful APIs
+++

* Paths of RESTful APIs follow a common pattern
* Paths can contain prefixes like `/api/` or `/api/v1/`
* The important part is a set of pairs of resource names and identifiers
* Deeper paths mean nesting of resources and subresources
* Examples are

    // Create a new customer.
    POST /api/v1/customers

    // Read a customer.
    GET /api/v1/customers/1

    // Read all addresses of a customer.
    GET /api/v1/customers/1/addresses

    // Read one address of a customer.
    GET /api/v1/customers/1/addresses/5

    // Update a customer.
    PUT /api/v1/customers/1

    // Delete a customer.
    DELETE /api/v1/customers/1

* The paths are unique identifiers of domain objects
* The pattern is `{prefix}/{resource}/{id}/{subresource}/{subresource-id}/...`
* The resource's name is always written in lowercase and plural

### Extending the package

```go
// Resource identifies a resource in a URI path by name and ID.
type Resource struct {
	Name string
	ID   string
}

// Resources is a number or resources in a URI path.
type Resources []Resource

// path returns the number of resource names concatenated with slashes
// like they are stored in the nested multiplexer.
func (ress Resources) path() string {
	names := make([]string, len(ress))
	for i, res := range ress {
		names[i] = res.Name
	}
	return strings.Join(names, "/")
}

// PathToResources parses a new Resource from a URI path.
func PathToResources(r *http.Request, prefix string) Resources {
	// Remove prefix with and without trailing slash.
	prefix = strings.TrimSuffix(prefix, "/")
	trimmed := strings.TrimPrefix(r.URL.Path, prefix)
	trimmed = strings.TrimPrefix(trimmed, "/")
	// Now split the path.
	parts := strings.Split(trimmed, "/")
	if len(parts) == 0 {
		return nil
	}
	var ress Resources
	var name string
	for i, part := range parts {
		switch {
		case i%2 == 0:
			name = part
		case i%2 == 1:
			ress = append(ress, Resource{name, part})
			name = ""
		}
	}
	if name != "" {
		ress = append(ress, Resource{name, ""})
	}
	return ress
}
```

---

[<< CRUD](crud.md) || [TABLE OF CONTENTS](../index.md) || [JSON >>](json.md)
