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
  - `kind` [required]
  - `apiVersion` [required]
  - `metadata` [required]
  - `spec` [varies] 
- Common to All Kinds: The Metadata
  - `name` [required]
  - `namespace` [optional]
  - `uid` [output]
  - `title` [optional]
  - `description` [optional]
  - `labels` [optional]
  - `annotations` [optional]
  - `tags` [optional]
  - `links` [optional] 
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
- The root fields `apiVersion`, `kind`, `metadata`, and `spec` are part of the envelope, defining the overall structure of all kinds of entity. Likewise, some metadata fields like `name`, `labels`, and `annotations` are of special significance and have reserved purposes and distinct shapes.

## Substitutions In The Descriptor Format
- The descriptor format supports substitutions using `$text`, `$json`, and `$yaml`.
- Placeholders like `$json: https://example.com/entity.json` are substituted by the content of the referenced file. Files can be referenced from any configured integration similar to locations by passing an absolute URL. It's also possible to reference relative files like `./referenced.yaml` from the same location. Relative references are handled relative to the folder of the `catalog-info.yaml` that contains the placeholder. There are three different types of placeholders:
  - `$text`: Interprets the contents of the referenced file as plain text and embeds it as a string.
  - `$json`: Interprets the contents of the referenced file as JSON and embeds the parsed structure.
  - `$yaml`: Interprets the contents of the referenced file as YAML and embeds the parsed structure.
- For example, this can be used to load the definition of an API entity from a web server and embed it as a string in the field `spec.definition`:
```
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: petstore
  description: The Petstore API
spec:
  type: openapi
  lifecycle: production
  owner: petstore@example.com
  definition:
    $text: https://petstore.swagger.io/v2/swagger.json
```
- Note that to be able to read from targets that are outside of the normal integration points such as github.com, you'll need to explicitly allow it by adding an entry in the backend.reading.allow list. Paths can be specified to further restrict targets For example:
```
backend:
  baseUrl: ...
  reading:
    allow:
      - host: example.com
      - host: '*.examples.org'
      - host: example.net
        paths: ['/api/']
```

## Common to All Kinds: The Envelope
1. kind [required]
2. apiVersion [required]
3. metadata [required]
4. spec [varies]

## `kind` [required]
- The `kind` is the high level entity type being described.
- The perhaps most central kind of entity, that the catalog focuses on in the initial phase, is `Component`.

### `apiVersion` [required]
- The `apiVersion` is the version of specification format for that particular entity that the specification is made against. The version is used for being able to evolve the format, and the tuple of `apiVersion` and `kind` should be enough for a parser to know how to interpret the contents of the rest of the data.
- Backstage specific entities have an `apiVersion` that is prefixed with `backstage.io/`, to distinguish them from other types of object that share the same type of structure. This may be relevant when co-hosting these specifications with e.g. Kubernetes object manifests, or when an organization adds their own specific kinds of entity to the catalog.
- Early versions of the catalog will be using alpha/beta versions, e.g. `backstage.io/v1alpha1`, to signal that the format may still change. After that, we will be using `backstage.io/v1` and up.

### `metadata` [required]
see below 👇
- A structure that contains metadata about the entity, i.e. things that aren't directly part of the entity specification itself.

### `spec` [varies]
- The actual specification data that describes the entity.
- The precise structure of the `spec` depends on the `apiVersion` and `kind` combination, and some kinds may not even have a spec at all. See further down in this document for the specification structure of specific kinds.

## Common to All Kinds: The Metadata
1. name [required]
2. namespace [optional]
3. uid [output]
4. title [optional]
5. description [optional]
6. labels [optional]
7. annotations [optional]
8. tags [optional]
9. links [optional]

- The metadata root field has a number of reserved fields with specific meaning, described below.
- In addition to these, you may add any number of other fields directly under metadata, but be aware that general plugins and tools may not be able to understand their semantics. See Extending the model for more information.

### `name` [required]
- The name of the entity. This name is both meant for human eyes to recognize the entity, and for machines and other components to reference the entity (e.g. in URLs or from other entity specification files).
- Names must be unique per kind, within a given namespace (if specified), at any point in time. This uniqueness constraint is case insensitive. Names may be reused at a later time, after an entity is deleted from the registry.
- Names are required to follow a certain format. Entities that do not follow those rules will not be accepted for registration in the catalog. The ruleset is configurable to fit your organization's needs, but the default behavior is as follows: 
  - Strings of length at least 1, and at most 63
  - Must consist of sequences of [a-z0-9A-Z] possibly separated by one of [-_.]
Example: `visits-tracking-service`, `CircleciBuildsDumpV2_avro_gcs`

### `namespace` [optional]
- For later use.
- The ID of a namespace that the entity belongs to. This field is optional, and currently has no special semantics apart from bounding the name uniqueness constraint if specified. It is reserved for future use and may get broader semantic implication later.
- For now, it is recommended to not specify a namespace unless you have specific need to do so. This means the entity belongs to the "default" namespace.
- Namespaces must be sequences of [a-zA-Z0-9], possibly separated by -, at most 63 characters in total. Namespace names are case insensitive and will be rendered as lower case in most places.
- Example: `tracking-services`, `payment`

### `uid` [output]
- Each entity gets an automatically generated globally unique ID when it first enters the database. This field is not meant to be specified as input data, but is rather created by the database engine itself when producing the output entity.

- Note that uid values are not to be seen as stable, and should not be used as external references to an entity. The uid can change over time even when a human observer might think that it wouldn't. As one of many examples, unregistering and re-registering the exact same file will result in a different uid value even though everything else is the same. Therefore there is very little, if any, reason to read or use this field externally.

- If you want to refer to an entity by some form of an identifier, you should always use string-form entity reference instead.

### `title` [optional]
- A display name of the entity, to be presented in user interfaces instead of the name property above, when available.

- This field is sometimes useful when the name is cumbersome or ends up being perceived as overly technical. The title generally does not have as stringent format requirements on it, so it may contain special characters and be more explanatory. Do keep it very short though, and avoid situations where a title can be confused with the name of another entity, or where two entities share a title.

- Note that this is only for display purposes, and may be ignored by some parts of the code. Entity references still always make use of the name property for example, not the title.

### `description` [optional]
- A human readable description of the entity, to be shown in Backstage. Should be kept short and informative, suitable to give an overview of the entity's purpose at a glance. More detailed explanations and documentation should be placed elsewhere.

### `labels` [optional]
- Labels are optional key/value pairs of that are attached to the entity, and their use is identical to Kubernetes object labels.
- Their main purpose is for references to other entities, and for information that is in one way or another classifying for the current entity. They are often used as values in queries or filters.
- Both the key and the value are strings, subject to the following restrictions.
- Keys have an optional prefix followed by a slash, and then the name part which is required. The prefix, if present, must be a valid lowercase domain name, at most 253 characters in total. The name part must be sequences of [a-zA-Z0-9] separated by any of [-_.], at most 63 characters in total.
- The backstage.io/ prefix is reserved for use by Backstage core components.
- Values are strings that follow the same restrictions as name above.

### `annotations` [optional]
- An object with arbitrary non-identifying metadata attached to the entity, identical in use to Kubernetes object annotations.
- Their purpose is mainly, but not limited, to reference into external systems. This could for example be a reference to the git ref the entity was ingested from, to monitoring and logging systems, to PagerDuty schedules, etc. Users may add these to descriptor YAML files, but in addition to this automated systems may also add annotations, either during ingestion into the catalog, or at a later time.
- Both the key and the value are strings, subject to the following restrictions.
- Keys have an optional prefix followed by a slash, and then the name part which is required. The prefix must be a valid lowercase domain name if specified, at most 253 characters in total. The name part must be sequences of [a-zA-Z0-9] separated by any of [-_.], at most 63 characters in total.
- The backstage.io/ prefix is reserved for use by Backstage core components.
- Values can be of any length, but are limited to being strings.
- There is a list of well-known annotations, but anybody is free to add more annotations as they see fit.

### `tags` [optional]
- A list of single-valued strings, for example to classify catalog entities in various ways. This is different to the labels in metadata, as labels are key-value pairs.
- The values are user defined, for example the programming language used for the component, like java or go.
- This field is optional, and currently has no special semantics.
- Each tag must be sequences of [a-z0-9:+#] separated by -, at most 63 characters in total.

### `links` [optional]
- A list of external hyperlinks related to the entity. Links can provide additional contextual information that may be located outside of Backstage itself. For example, an admin dashboard or external CMS page.
- Users may add links to descriptor YAML files to provide additional reference information to external content & resources. Links are not intended to drive any additional functionality within Backstage, which is best left to annotations and labels. It is recommended to use links only when an equivalent well-known annotation does not cover a similar use case.
- Fields of a link are:

| Field |  Type  |                                   Description                                  |
|:-----:|:------:|:------------------------------------------------------------------------------:|
| url   | String | [Required] A url in a standard uri format (e.g. https://example.com/some/page) |
| title | String | [Optional] A user friendly display name for the link. |
| icon | String | [Optional] A key representing a visual icon to be displayed in the UI. |
| type |  String | [Optional] An optional value to categorize links into specific groups. |

- NOTE: The icon field value is meant to be a semantic key that will map to a specific icon that may be provided by an icon library (e.g. material-ui icons). These keys should be a sequence of [a-z0-9A-Z], possibly separated by one of [-_.]. Backstage may support some basic icons out of the box such as those defined in app-defaults, but the Backstage integrator will ultimately be left to provide the appropriate icon component mappings. A generic fallback icon would be provided if a mapping cannot be resolved.
- The semantics of the type field are undefined. The adopter is free to define their own set of types and utilize them as they wish. Some potential use cases can be to utilize the type field to validate certain links exist on entities or to create customized UI components for specific link types.

## Common to All Kinds: Relations
## Common to All Kinds: Status
## Kind: Component
## Kind: Template
## Kind: API
## Kind: Group
## Kind: User
## Kind: Resource
## Kind: System
## Kind: Domain
## Kind: Location

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
