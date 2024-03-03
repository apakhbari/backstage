# Descriptor Format of Catalog Entities
```
 __   __  _______  __   __  ___          _______  ___   ___      _______      _______  _______  ______    __   __  _______  _______ 
|  | |  ||   _   ||  |_|  ||   |        |       ||   | |   |    |       |    |       ||       ||    _ |  |  |_|  ||   _   ||       |
|  |_|  ||  |_|  ||       ||   |        |    ___||   | |   |    |    ___|    |    ___||   _   ||   | ||  |       ||  |_|  ||_     _|
|       ||       ||       ||   |        |   |___ |   | |   |    |   |___     |   |___ |  | |  ||   |_||_ |       ||       |  |   |  
|_     _||       ||       ||   |___     |    ___||   | |   |___ |    ___|    |    ___||  |_|  ||    __  ||       ||       |  |   |  
  |   |  |   _   || ||_|| ||       |    |   |    |   | |       ||   |___     |   |    |       ||   |  | || ||_|| ||   _   |  |   |  
  |___|  |__| |__||_|   |_||_______|    |___|    |___| |_______||_______|    |___|    |_______||___|  |_||_|   |_||__| |__|  |___|  
```

## Index:
- Overall Shape Of An Entity
- Common to All Kinds: The Envelope
- Common to All Kinds: The Metadata
- Common to All Kinds: Relations
- Common to All Kinds: Status
- Kind: Component
- Kind: Template
- Kind: API
- Kind: Group
- Kind: User
- Kind: Resource
- Kind: System
- Kind: Domain
- Kind: Location

## Overall Shape Of An Entity
- The following is an example of a descriptor file for a Component entity:
```
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: artist-web
  description: The place to be, for great artists
  labels:
    example.com/custom: custom_label_value
  annotations:
    example.com/service-discovery: artistweb
    circleci.com/project-slug: github/example-org/artist-website
  tags:
    - java
  links:
    - url: https://admin.example-org.com
      title: Admin Dashboard
      icon: dashboard
      type: admin-dashboard
spec:
  type: website
  lifecycle: production
  owner: artist-relations-team
  system: public-websites
```

- This is the same entity as returned in JSON from the software catalog API:
```
{
  "apiVersion": "backstage.io/v1alpha1",
  "kind": "Component",
  "metadata": {
    "annotations": {
      "backstage.io/managed-by-location": "file:/tmp/catalog-info.yaml",
      "example.com/service-discovery": "artistweb",
      "circleci.com/project-slug": "github/example-org/artist-website"
    },
    "description": "The place to be, for great artists",
    "etag": "ZjU2MWRkZWUtMmMxZS00YTZiLWFmMWMtOTE1NGNiZDdlYzNk",
    "labels": {
      "example.com/custom": "custom_label_value"
    },
    "links": [
      {
        "url": "https://admin.example-org.com",
        "title": "Admin Dashboard",
        "icon": "dashboard",
        "type": "admin-dashboard"
      }
    ],
    "tags": ["java"],
    "name": "artist-web",
    "uid": "2152f463-549d-4d8d-a94d-ce2b7676c6e2"
  },
  "spec": {
    "lifecycle": "production",
    "owner": "artist-relations-team",
    "type": "website",
    "system": "public-websites"
  }
}
```

# acknowledgment
## Contributors

APA 🖖🏻

## Links

```
  aaaaaaaaaaaaa  ppppp   ppppppppp     aaaaaaaaaaaaa   
  a::::::::::::a p::::ppp:::::::::p    a::::::::::::a  
  aaaaaaaaa:::::ap:::::::::::::::::p   aaaaaaaaa:::::a 
           a::::app::::::ppppp::::::p           a::::a 
    aaaaaaa:::::a p:::::p     p:::::p    aaaaaaa:::::a 
  aa::::::::::::a p:::::p     p:::::p  aa::::::::::::a 
 a::::aaaa::::::a p:::::p     p:::::p a::::aaaa::::::a 
a::::a    a:::::a p:::::p    p::::::pa::::a    a:::::a 
a::::a    a:::::a p:::::ppppp:::::::pa::::a    a:::::a 
a:::::aaaa::::::a p::::::::::::::::p a:::::aaaa::::::a 
 a::::::::::aa:::ap::::::::::::::pp   a::::::::::aa:::a
  aaaaaaaaaa  aaaap::::::pppppppp      aaaaaaaaaa  aaaa
                  p:::::p                              
                  p:::::p                              
                 p:::::::p                             
                 p:::::::p                             
                 p:::::::p                             
                 ppppppppp                             
```
