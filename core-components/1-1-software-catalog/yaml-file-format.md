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
  - ownedBy and ownerOf
  - providesApi and apiProvidedBy
  - consumesApi and apiConsumedBy
  - dependsOn and dependencyOf
  - parentOf and childOf
  - memberOf and hasMember
  - partOf and hasPart
  - Common to All Kinds: Status
[Divider] [Divider] [Divider] [Divider]
- Kind: `Component`
  - `apiVersion` and `kind` [required]
  - `spec.type` [required]
  - `spec.lifecycle` [required]
  - `spec.owner` [required]
  - `spec.system` [optional]
  - `spec.subcomponentOf` [optional]
  - `spec.providesApis` [optional]
  - `spec.consumesApis` [optional]
  - `spec.dependsOn` [optional]
- Kind: `Template`
  - `apiVersion` and `kind` [required]
  - `metadata.tags` [optional]
  - `spec.type` [required]
  - `spec.parameters` [required]
  - `spec.steps` [required]
  - `spec.owner` [optional] 
- Kind: `API`
  - `apiVersion` and `kind` [required]
  - `spec.type` [required]
  - `spec.lifecycle` [required]
  - `spec.owner` [required]
  - `spec.system` [optional]
  - `spec.definition` [required]
- Kind: `Group`
  - `apiVersion` and `kind` [required]
  - `spec.type` [required]
  - `spec.profile` [optional]
  - `spec.parent` [optional]
  - `spec.children` [required]
  - `spec.members` [optional] 
- Kind: `User`
  - `apiVersion` and `kind` [required]
  - `spec.profile` [optional]
  - `spec.memberOf` [required]
- Kind: `Resource`
  - `apiVersion` and `kind` [required]
  - `spec.owner` [required]
  - `spec.type` [required]
  - `spec.system` [optional]
  - `spec.dependsOn` [optional]
  - `spec.dependencyOf` [optional]
- Kind: `System`
  - `apiVersion` and `kind` [required]
  - `spec.owner` [required]
  - `spec.domain` [optional]
- Kind: `Domain`
  - `apiVersion` and `kind` [required]
  - `spec.owner` [required]
- Kind: `Location`
  - `apiVersion` and `kind` [required]
  - `spec` [required]
  - `spec.type` [optional]
  - `spec.target` [optional]
  - `spec.targets` [optional]
  - `spec.presence` [optional]

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

#### Well known Annotations
see: https://backstage.io/docs/features/software-catalog/well-known-annotations

- backstage.io/managed-by-location
- backstage.io/managed-by-origin-location

- backstage.io/orphan

- backstage.io/techdocs-ref
- backstage.io/techdocs-entity

- backstage.io/view-url, backstage.io/edit-url

- backstage.io/source-location

- jenkins.io/job-full-name

- github.com/project-slug
- github.com/team-slug
- github.com/user-login

- gocd.org/pipelines

- periskop.io/service-name

- sentry.io/project-slug

- rollbar.com/project-slug

- circleci.com/project-slug

- backstage.io/ldap-rdn, backstage.io/ldap-uuid, backstage.io/ldap-dn

- graph.microsoft.com/tenant-id, graph.microsoft.com/group-id, graph.microsoft.com/user-id

- sonarqube.org/project-key

- backstage.io/code-coverage

- vault.io/secrets-path

##### Deprecated Annotations
- backstage.io/github-actions-id
- backstage.io/definition-at-location
- jenkins.io/github-folder


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
- The relations root field is a read-only list of relations, between the current entity and other entities, described in the well-known relations section. Relations are commonly two-way, so that there's a pair of relation types each describing one direction of the relation.
- A relation as part of a single entity that's read out of the API may look as follows.
```
{
  // ...
  "relations": [
    {
      "type": "ownedBy",
      "targetRef": "group:default/dev.infra"
    }
  ],
  "spec": {
    "owner": "dev.infra",
    // ...
  }
}
```
- The fields of a relation are:

|   Field   |  Type  |                           Description                           |
|:---------:|:------:|:---------------------------------------------------------------:|
| targetRef | String | A full   entity reference  to the other end of the relation.    |
|    type   | String | The type of relation FROM a source entity TO the target entity. |
| metadata  | Object | Reserved for future use.                                        |

- Entity descriptor YAML files are not supposed to contain this field. Instead, catalog processors analyze the entity descriptor data and its surroundings, and deduce relations that are then attached onto the entity as read from the catalog.
- Where relations are produced, they are to be considered the authoritative source for that piece of data. In the example above, a plugin would do better to consume the relation rather than spec.owner for deducing the owner of the entity, because it may even be the case that the owner isn't taken from the YAML at all - it could be taken from a CODEOWNERS file nearby instead for example. Also, the spec.owner is on a shortened form and may have semantics associated with it (such as the default kind being Group if not specified).

- Each relation has a source (implicitly: the entity that holds the relation), a target (the entity to which the source has a relation), and a type that tells what relation the source has with the target. The relation is directional; there are commonly pairs of relation types and the entity at the other end will have the opposite relation in the opposite direction (e.g. when querying for `A`, you will see `A.ownedBy.B`, and when querying `B`, you will see `B.ownerOf.A`).

### Well known Relations
see: https://backstage.io/docs/features/software-catalog/well-known-relations

#### ownedBy and ownerOf
- An ownership relation where the owner is usually an organizational entity (`User` or `Group`), and the other entity can be anything.

#### providesApi and apiProvidedBy
- A relation with an `API` entity, typically from a `Component`.

#### consumesApi and apiConsumedBy
- A relation with an `API` entity, typically from a `Component`.

#### dependsOn and dependencyOf
- A relation denoting a dependency on another entity.

#### parentOf and childOf
- A parent/child relation to build up a tree, used for example to describe the organizational structure between `Groups`.

#### memberOf and hasMember
- A membership relation, typically for `Users` in `Groups`.

#### partOf and hasPart
- A relation with a `Domain`, `System` or `Component` entity, typically from a `Component`, `API`, or `System`.


## Common to All Kinds: Status
- IMPORTANT: This status is in active development and its format will change unexpectedly. Do not consume it in your own code until such a time that this documentation has been updated.

- The status root object is a read-only set of statuses, pertaining to the current state or health of the entity, described in the well-known statuses section.
- Currently, the only defined field is the items array. Each of its items contains a specific data structure that describes some aspect of the state of the entity, as seen from the point of view of some specific system. Different systems may contribute to this array, under their own respective type keys.
- The current main use case for this field is for the ingestion processes of the catalog itself to convey information about errors and warnings back to the user.
- A status field as part of a single entity that's read out of the API may look as follows:
```
{
  // ...
  "status": {
    "items": [
      {
        "type": "backstage.io/catalog-processing",
        "level": "error",
        "message": "NotFoundError: File not found",
        "error": {
          "name": "NotFoundError",
          "message": "File not found",
          "stack": "..."
        }
      }
    ]
  },
  "spec": {
    // ...
  }
}
```
- The fields of a status item are:
|  Field  |  Type  |                                            Description                                           |
|:-------:|:------:|:------------------------------------------------------------------------------------------------:|
| type    | String | The type of status as a unique key per source. Each type may appear more than once in the array. |
|  level  | String |              The level / severity of the status item: 'info', 'warning, or 'error'.              |
| message | String | A brief message describing the status, intended for human consumption.                           |
| error   | Object | An optional serialized error object related to the status.                                       |

- The `type` is an arbitrary string, but we recommend that types that are not strictly private within the organization be namespaced to avoid collisions. Types emitted by Backstage core processes will for example be prefixed with `backstage.io/` as in the example above.
- Entity descriptor YAML files are not supposed to contain a `status` root key. Instead, catalog processors analyze the entity descriptor data and its surroundings, and deduce status entries that are then attached onto the entity as read from the catalog.
- See the well-known statuses section for a list of well-known / common status types.

---
---

## Kind: Component
Describes the following entity kind:

|    Field   |         Value         |
|:----------:|:---------------------:|
| apiVersion | backstage.io/v1alpha1 |
| kind       | Component             |

- A Component describes a software component. It is typically intimately linked to the source code that constitutes the component, and should be what a developer may regard a "unit of software", usually with a distinct deployable or linkable artifact.

- Descriptor files for this kind may look as follows:
```
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: artist-web
  description: The place to be, for great artists
spec:
  type: website
  lifecycle: production
  owner: artist-relations-team
  system: artist-engagement-portal
  dependsOn:
    - resource:default/artists-db
  providesApis:
    - artist-api
```

this kind has the following structure:

### `apiVersion` and `kind` [required]
- Exactly equal to `backstage.io/v1alpha1` and `Component`, respectively.

### `spec.type` [required]
- The type of component as a string, e.g. `website`. This field is required.
- The software catalog accepts any type value, but an organization should take great care to establish a proper taxonomy for these. Tools including Backstage itself may read this field and behave differently depending on its value. For example, a website type component may present tooling in the Backstage interface that is specific to just websites.
- The current set of well-known and common values for this field is:

1. `service` - a backend service, typically exposing an API
2. `website` - a website
3. `library` - a software library, such as an npm module or a Java library

### `spec.lifecycle` [required]
- The lifecycle state of the component, e.g. `production`. This field is required.
- The software catalog accepts any lifecycle value, but an organization should take great care to establish a proper taxonomy for these.
- The current set of well-known and common values for this field is:

1. `experimental` - an experiment or early, non-production component, signaling that users may not prefer to consume it over other more established components, or that there are low or no reliability guarantees
2. `production` - an established, owned, maintained component
3. `deprecated` - a component that is at the end of its lifecycle, and may disappear at a later point in time

### `spec.owner` [required]
- An entity reference to the owner of the component, e.g. artist-relations-team. This field is required.
- In Backstage, the owner of a component is the singular entity (commonly a team) that bears ultimate responsibility for the component, and has the authority and capability to develop and maintain it. They will be the point of contact if something goes wrong, or if features are to be requested. The main purpose of this field is for display purposes in Backstage, so that people looking at catalog items can get an understanding of to whom this component belongs. It is not to be used by automated processes to for example assign authorization in runtime systems. There may be others that also develop or otherwise touch the component, but there will always be one ultimate owner.

|            `kind`           |            Default [`namespace`]           | Generated `relation` type         |
|:-------------------------:|:----------------------------------------:|:---------------------------------:|
| `Group` (default), `User` | Same as this entity, typically `default` | `ownerOf` , and reverse   `ownedBy` |

### `spec.system` [optional]
- An entity reference to the system that the component belongs to, e.g. `artist-engagement-portal`. This field is optional.

|       kind       |           Default [namespace]          | Generated relation type     |
|:----------------:|:--------------------------------------:|:-----------------------------:|
| System (default) | Same as this entity, typically default | partOf, and reverse hasPart |

### `spec.subcomponentOf` [optional]
- An entity reference to another component of which the component is a part, e.g. `spotify-ios-app`. This field is optional.

|         kind        |           Default [namespace]          | Generated relation type     |
|:-------------------:|:--------------------------------------:|:-----------------------------:|
| component (default) | Same as this entity, typically default | partOf, and reverse hasPart |

### `spec.providesApis` [optional]
- An array of entity references to the APIs that are provided by the component, e.g. `artist-api`. This field is optional.

|      kind     |           Default [namespace]          | Generated relation type     |
|:-------------:|:--------------------------------------:|:-----------------------------:|
| API (default) | Same as this entity, typically default | partOf, and reverse hasPart |

### `spec.consumesApis` [optional]
- An array of entity references to the APIs that are consumed by the component, e.g. `artist-api`. This field is optional.

|      kind     |           Default [namespace]          | Generated relation type     |
|:-------------:|:--------------------------------------:|:-----------------------------:|
| API (default) | Same as this entity, typically default | partOf, and reverse hasPart |

### `spec.dependsOn` [optional]
- An array of entity references to the components and resources that the component depends on, e.g. `artists-db`. This field is optional.

|    kind   |           Default [namespace]          | Generated relation type             |
|:---------:|:--------------------------------------:|:-------------------------------------:|
| Component | Same as this entity, typically default | dependsOn, and reverse dependencyOf |
| Resource  | Same as this entity, typically default | dependsOn, and reverse dependencyOf |

## Kind: Template
Describes the following entity kind:

|    Field   |         Value         |
|:----------:|:---------------------:|
| apiVersion | backstage.io/v1beta2  |
| kind       | Template              |

- A template definition describes both the parameters that are rendered in the frontend part of the scaffolding wizard, and the steps that are executed when scaffolding that component.

- Descriptor files for this kind may look as follows.
```
apiVersion: backstage.io/v1beta2
kind: Template
# some metadata about the template itself
metadata:
  name: v1beta2-demo
  title: Test Action template
  description: scaffolder v1beta2 template demo
spec:
  owner: backstage/techdocs-core
  type: service

  # these are the steps which are rendered in the frontend with the form input
  parameters:
    - title: Fill in some steps
      required:
        - name
      properties:
        name:
          title: Name
          type: string
          description: Unique name of the component
          ui:autofocus: true
          ui:options:
            rows: 5
    - title: Choose a location
      required:
        - repoUrl
      properties:
        repoUrl:
          title: Repository Location
          type: string
          ui:field: RepoUrlPicker
          ui:options:
            allowedHosts:
              - github.com

  # here's the steps that are executed in series in the scaffolder backend
  steps:
    - id: fetch-base
      name: Fetch Base
      action: fetch:template
      input:
        url: ./template
        values:
          name: '{{ parameters.name }}'

    - id: fetch-docs
      name: Fetch Docs
      action: fetch:plain
      input:
        targetPath: ./community
        url: https://github.com/backstage/community/tree/main/backstage-community-sessions

    - id: publish
      name: Publish
      action: publish:github
      input:
        allowedHosts: ['github.com']
        description: 'This is {{ parameters.name }}'
        repoUrl: '{{ parameters.repoUrl }}'

    - id: register
      name: Register
      action: catalog:register
      input:
        repoContentsUrl: {{ steps['publish'].output.repoContentsUrl }}
        catalogInfoPath: '/catalog-info.yaml'
```

this kind has the following structure:

### `apiVersion` and `kind` [required]
- Exactly equal to `backstage.io/v1beta2` and `Template`, respectively.

### `metadata.tags` [optional]
- A list of strings that can be associated with the template, e.g. [`recommended`, `react`].
- This list will also be used in the frontend to display to the user so you can potentially search and group templates by these tags.

### `spec.type` [required]
- The type of component created by the template, e.g. `website`. This is used for filtering templates, and should ideally match the Component `spec.type` created by the template.

### `spec.parameters` [required]
- You can find out more about this in template section

### `spec.steps` [required]
- You can find out more about this in template section

### `spec.owner` [optional]
- An entity reference to the owner of the template, e.g. `artist-relations-team`. This field is required.
- In Backstage, the owner of a Template is the singular entity (commonly a team) that bears ultimate responsibility for the Template, and has the authority and capability to develop and maintain it. They will be the point of contact if something goes wrong, or if features are to be requested. The main purpose of this field is for display purposes in Backstage, so that people looking at catalog items can get an understanding of to whom this Template belongs. It is not to be used by automated processes to for example assign authorization in runtime systems. There may be others that also develop or otherwise touch the Template, but there will always be one ultimate owner.

|            `kind`           |            Default [`namespace`]           | Generated `relation` type         |
|:-------------------------:|:----------------------------------------:|:---------------------------------:|
| `Group` (default), `User` | Same as this entity, typically `default` | `ownerOf` , and reverse   `ownedBy` |

## Kind: API
Describes the following entity kind:

|    Field   |         Value         |
|:----------:|:---------------------:|
| apiVersion | backstage.io/v1alpha1 |
| kind       | API                   |

- An API describes an interface that can be exposed by a component. The API can be defined in different formats, like OpenAPI, AsyncAPI, GraphQL, gRPC, or other formats.

Descriptor files for this kind may look as follows:

```
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: artist-api
  description: Retrieve artist details
spec:
  type: openapi
  lifecycle: production
  owner: artist-relations-team
  system: artist-engagement-portal
  definition: |
    openapi: "3.0.0"
    info:
      version: 1.0.0
      title: Artist API
      license:
        name: MIT
    servers:
      - url: http://artist.spotify.net/v1
    paths:
      /artists:
        get:
          summary: List all artists
    ...
```

this kind has the following structure:

### `apiVersion` and `kind` [required]
- Exactly equal to `backstage.io/v1alpha1` and `API`, respectively.

### `spec.type` [required]
- The type of the API definition as a string, e.g. `openapi`. This field is required.
- The software catalog accepts any type value, but an organization should take great care to establish a proper taxonomy for these. Tools including Backstage itself may read this field and behave differently depending on its value. For example, an OpenAPI type API may be displayed using an OpenAPI viewer tooling in the Backstage interface.
- The current set of well-known and common values for this field is:
  - `openapi` - An API definition in YAML or JSON format based on the OpenAPI version 2 or version 3 spec.
  - `asyncapi` - An API definition based on the AsyncAPI version 2 or version 3 spec.
  - `graphql` - An API definition based on GraphQL schemas for consuming GraphQL based APIs.
  - `grpc` - An API definition based on Protocol Buffers to use with gRPC.

### `spec.lifecycle` [required]
- The lifecycle state of the component, e.g. `production`. This field is required.
- The software catalog accepts any lifecycle value, but an organization should take great care to establish a proper taxonomy for these.
- The current set of well-known and common values for this field is:

1. `experimental` - an experiment or early, non-production component, signaling that users may not prefer to consume it over other more established components, or that there are low or no reliability guarantees
2. `production` - an established, owned, maintained component
3. `deprecated` - a component that is at the end of its lifecycle, and may disappear at a later point in time

### `spec.owner` [required]
- An entity reference to the owner of the component, e.g. artist-relations-team. This field is required.
- In Backstage, the owner of a component is the singular entity (commonly a team) that bears ultimate responsibility for the component, and has the authority and capability to develop and maintain it. They will be the point of contact if something goes wrong, or if features are to be requested. The main purpose of this field is for display purposes in Backstage, so that people looking at catalog items can get an understanding of to whom this component belongs. It is not to be used by automated processes to for example assign authorization in runtime systems. There may be others that also develop or otherwise touch the component, but there will always be one ultimate owner.

|            `kind`           |            Default [`namespace`]           | Generated `relation` type         |
|:-------------------------:|:----------------------------------------:|:---------------------------------:|
| `Group` (default), `User` | Same as this entity, typically `default` | `ownerOf` , and reverse   `ownedBy` |

### `spec.system` [optional]
- An entity reference to the system that the component belongs to, e.g. `artist-engagement-portal`. This field is optional.

|       kind       |           Default [namespace]          | Generated relation type     |
|:----------------:|:--------------------------------------:|:-----------------------------:|
| System (default) | Same as this entity, typically default | partOf, and reverse hasPart |

### `spec.definition` [required]
- The definition of the API, based on the format defined by spec.type. This field is required.

## Kind: Group
Describes the following entity kind:

|    Field   |         Value         |
|:----------:|:---------------------:|
| apiVersion | backstage.io/v1alpha1 |
| kind       | Group                 |

- A group describes an organizational entity, such as for example a team, a business unit, or a loose collection of people in an interest group. Members of these groups are modeled in the catalog as kind `User`.

- Descriptor files for this kind may look as follows:
```
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  name: infrastructure
  description: The infra business unit
spec:
  type: business-unit
  profile:
    displayName: Infrastructure
    email: infrastructure@example.com
    picture: https://example.com/groups/bu-infrastructure.jpeg
  parent: ops
  children: [backstage, other]
  members: [jdoe]
```
this kind has the following structure:

### `apiVersion` and `kind` [required]
- Exactly equal to `backstage.io/v1alpha1` and `Group`, respectively.

### `spec.type` [required]
- The type of group as a string, e.g. team. There is currently no enforced set of values for this field, so it is left up to the adopting organization to choose a nomenclature that matches their org hierarchy.

- Some common values for this field could be:
  - `team`
  - `business-unit`
  - `product-area`
  - `root` - as a common virtual root of the hierarchy, if desired
 
### `spec.profile` [optional]
- Optional profile information about the group, mainly for **display purposes**. All fields of this structure are also optional. The email would be a group email of some form, that the group may wish to be used for contacting them. The picture is expected to be a URL pointing to an image that's representative of the group, and that a browser could fetch and render on a group page or similar.

The fields of a profile are:
|          Field         |  Type  |                           Description                          |
|:----------------------:|:------:|:--------------------------------------------------------------:|
| displayName (optional) | String |              A human-readable name for the group.              |
| email (optional)       | String |   An email the group may wish to be used for contacting them.  |
| picture (optional)     | String | A URL pointing to an image that's representative of the group. |

### `spec.parent` [optional]
- The immediate parent group in the hierarchy, if any. Not all groups must have a parent; the catalog supports multi-root hierarchies. Groups may however not have more than one parent.

- This field is an `entity reference`.

|       kind      |            Default namespace           |    Generated relation type    |
|:---------------:|:--------------------------------------:|:-----------------------------:|
| Group (default) | Same as this entity, typically default | childOf, and reverse parentOf |

### `spec.children` [required]
- The immediate child groups of this group in the hierarchy (whose parent field points to this group). The list must be present, but may be empty if there are no child groups. The items are not guaranteed to be ordered in any particular way.

- The entries of this array are entity references.

|       kind      |            Default namespace           |    Generated relation type    |
|:---------------:|:--------------------------------------:|:-----------------------------:|
| Group (default) | Same as this entity, typically default | childOf, and reverse parentOf |

### `spec.members` [optional]
- The users that are direct members of this group. The items are not guaranteed to be ordered in any particular way.

- The entries of this array are entity references.

|       kind      |            Default namespace           |    Generated relation type    |
|:---------------:|:--------------------------------------:|:-----------------------------:|
| User (default)  | Same as this entity, typically default | hasMember, and reverse memberOf |

## Kind: User
Describes the following entity kind:

|    Field   |         Value         |
|:----------:|:---------------------:|
| apiVersion | backstage.io/v1alpha1 |
| kind       | User                  |

- A user describes a person, such as an employee, a contractor, or similar. Users belong to Group entities in the catalog.
- These catalog user entries are connected to the way that authentication within the Backstage ecosystem works. See the auth section of the docs for a discussion of these concepts.

- Descriptor files for this kind may look as follows.
```
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: jdoe
spec:
  profile:
    displayName: Jenny Doe
    email: jenny-doe@example.com
    picture: https://example.com/staff/jenny-with-party-hat.jpeg
  memberOf: [team-b, employees]
```
this kind has the following structure:

### `apiVersion` and `kind` [required]
- Exactly equal to `backstage.io/v1alpha1` and `User`, respectively.

### `spec.profile` [optional]
- Optional profile information about the user, mainly for display purposes. All fields of this structure are also optional. The email would be a primary email of some form, that the user may wish to be used for contacting them. The picture is expected to be a URL pointing to an image that's representative of the user, and that a browser could fetch and render on a profile page or similar.

- The fields of a profile are:
|          Field         |  Type  |                           Description                          |
|:----------------------:|:------:|:--------------------------------------------------------------:|
| displayName (optional) | String |              A human-readable name for the user.              |
| email (optional)       | String |   An email the user may wish to be used for contacting them.  |
| picture (optional)     | String | A URL pointing to an image that's representative of the user. |

### `spec.memberOf` [required]
- The list of groups that the user is a direct member of (i.e., no transitive memberships are listed here). The list must be present, but may be empty if the user is not member of any groups. The items are not guaranteed to be ordered in any particular way.

- The entries of this array are entity references.
|       kind      |            Default namespace           |    Generated relation type    |
|:---------------:|:--------------------------------------:|:-----------------------------:|
| Group (default)  | Same as this entity, typically default | memberOf, and reverse hasMember |

## Kind: Resource
Describes the following entity kind:

|    Field   |         Value         |
|:----------:|:---------------------:|
| apiVersion | backstage.io/v1alpha1 |
| kind       | Resource              |

- A resource describes the infrastructure a system needs to operate, like BigTable databases, Pub/Sub topics, S3 buckets or CDNs. Modelling them together with components and systems allows to visualize resource footprint, and create tooling around them.

- The main purpose of this field is for display purposes in Backstage

- Descriptor files for this kind may look as follows:
```
apiVersion: backstage.io/v1alpha1
kind: Resource
metadata:
  name: artists-db
  description: Stores artist details
spec:
  type: database
  owner: artist-relations-team
  system: artist-engagement-portal
```
this kind has the following structure:

### `apiVersion` and `kind` [required]
- Exactly equal to backstage.io/v1alpha1 and Resource, respectively.

### `spec.owner` [required]
- An entity reference to the owner of the resource, e.g. `artist-relations-team`. This field is required.
- In Backstage, the owner of a resource is the singular entity (commonly a team) that bears ultimate responsibility for the resource, and has the authority and capability to develop and maintain it. They will be the point of contact if something goes wrong, or if features are to be requested. **The main purpose of this field is for display purposes in Backstage**, so that people looking at catalog items can get an understanding of to whom this resource belongs. It is not to be used by automated processes to for example assign authorization in runtime systems. There may be others that also manage or otherwise touch the resource, but there will always be one ultimate owner.

|       kind      |            Default namespace           |    Generated relation type    |
|:---------------:|:--------------------------------------:|:-----------------------------:|
| Group (default)  | Same as this entity, typically default | ownerOf, and reverse ownedBy |

### `spec.type` [required]
- The type of resource as a string, e.g. database. This field is required. There is currently no enforced set of values for this field, so it is left up to the adopting organization to choose a nomenclature that matches the resources used in their tech stack.

- Some common values for this field could be:
1. `database`
2. `s3-bucket`
3. `kubernetes-cluster`

### `spec.system` [optional]
- An entity reference to the system that the resource belongs to, e.g. `artist-engagement-portal`. This field is optional.


|       kind       |           Default [namespace]          | Generated relation type     |
|:----------------:|:--------------------------------------:|:-----------------------------:|
| System (default) | Same as this entity, typically default | partOf, and reverse hasPart |


### `spec.dependsOn` [optional]
- An array of entity references to the components and resources that the component depends on, e.g. `artists-lookup`. This field is optional.

|    kind   |           Default [namespace]          | Generated relation type             |
|:---------:|:--------------------------------------:|:-------------------------------------:|
| Component | Same as this entity, typically default | dependsOn, and reverse dependencyOf |
| Resource  | Same as this entity, typically default | dependsOn, and reverse dependencyOf |

### `spec.dependencyOf` [optional]
- An array of entity references to the components and resources that the resource is a dependency of, e.g. `artist-lookup`. This field is optional.

|    kind   |           Default [namespace]          | Generated relation type             |
|:---------:|:--------------------------------------:|:-------------------------------------:|
| Component | Same as this entity, typically default | dependsOn, and reverse dependencyOf |
| Resource  | Same as this entity, typically default | dependsOn, and reverse dependencyOf |

## Kind: System
Describes the following entity kind:

|    Field   |         Value         |
|:----------:|:---------------------:|
| apiVersion | backstage.io/v1alpha1 |
| kind       | System                |

- A system is a collection of resources and components. The system may expose or consume one or several APIs. It is viewed as abstraction level that provides potential consumers insights into exposed features without needing a too detailed view into the details of all components. This also gives the owning team the possibility to decide about published artifacts and APIs.
- Descriptor files for this kind may look as follows:
```
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: artist-engagement-portal
  description: Handy tools to keep artists in the loop
spec:
  owner: artist-relations-team
  domain: artists
```

this kind has the following structure:

### `apiVersion` and `kind` [required]
- Exactly equal to `backstage.io/v1alpha1` and `System`, respectively.

### `spec.owner` [required]
- An entity reference to the owner of the resource, e.g. `artist-relations-team`. This field is required.
- In Backstage, the owner of a resource is the singular entity (commonly a team) that bears ultimate responsibility for the resource, and has the authority and capability to develop and maintain it. They will be the point of contact if something goes wrong, or if features are to be requested. **The main purpose of this field is for display purposes in Backstage**, so that people looking at catalog items can get an understanding of to whom this resource belongs. It is not to be used by automated processes to for example assign authorization in runtime systems. There may be others that also manage or otherwise touch the resource, but there will always be one ultimate owner.

|       kind      |            Default namespace           |    Generated relation type    |
|:---------------:|:--------------------------------------:|:-----------------------------:|
| Group (default)  | Same as this entity, typically default | ownerOf, and reverse ownedBy |

### `spec.domain` [optional]
- An entity reference to the domain that the system belongs to, e.g. `artists`. This field is optional.

|       kind       |            Default namespace           |   Generated relation type   |
|:----------------:|:--------------------------------------:|:---------------------------:|
| Domain (default) | Same as this entity, typically default | partOf, and reverse hasPart |

## Kind: Domain
Describes the following entity kind:

|    Field   |         Value         |
|:----------:|:---------------------:|
| apiVersion | backstage.io/v1alpha1 |
| kind       | Domain                |

- A Domain groups a collection of systems that share terminology, domain models, business purpose, or documentation, i.e. form a bounded context.

- Descriptor files for this kind may look as follows:
```
apiVersion: backstage.io/v1alpha1
kind: Domain
metadata:
  name: artists
  description: Everything about artists
spec:
  owner: artist-relations-team
```
- this kind has the following structure:

### `apiVersion` and `kind` [required]
- Exactly equal to `backstage.io/v1alpha1` and `System`, respectively.

### `spec.owner` [required]
- An entity reference to the owner of the resource, e.g. `artist-relations-team`. This field is required.
- In Backstage, the owner of a resource is the singular entity (commonly a team) that bears ultimate responsibility for the resource, and has the authority and capability to develop and maintain it. They will be the point of contact if something goes wrong, or if features are to be requested. **The main purpose of this field is for display purposes in Backstage**, so that people looking at catalog items can get an understanding of to whom this resource belongs. It is not to be used by automated processes to for example assign authorization in runtime systems. There may be others that also manage or otherwise touch the resource, but there will always be one ultimate owner.

|       kind      |            Default namespace           |    Generated relation type    |
|:---------------:|:--------------------------------------:|:-----------------------------:|
| Group (default)  | Same as this entity, typically default | ownerOf, and reverse ownedBy |

## Kind: Location
Describes the following entity kind:

|    Field   |         Value         |
|:----------:|:---------------------:|
| apiVersion | backstage.io/v1alpha1 |
| kind       | Location              |

- A location is a marker that references other places to look for catalog data.

Descriptor files for this kind may look as follows.
```
apiVersion: backstage.io/v1alpha1
kind: Location
metadata:
  name: org-data
spec:
  type: url
  targets:
    - http://github.com/myorg/myproject/org-data-dump/catalog-info-staff.yaml
    - http://github.com/myorg/myproject/org-data-dump/catalog-info-consultants.yaml
```
This kind has the following structure:

### `apiVersion` and `kind` [required]
- Exactly equal to backstage.io/v1alpha1 and Location, respectively.

### `spec` [required]
- The `spec` field is required. The minimal spec should be an empty object.

### `spec.type` [optional]
- The single location type, that's common to the targets specified in the spec. If it is left out, it is inherited from the location type that originally read the entity data. For example, if you have a `url` type location, that when read results in a `Location` kind entity with no `spec.type`, then the referenced targets in the entity will implicitly also be of `url` type. This is useful because you can define a hierarchy of things in a directory structure using relative target paths (see below), and it will work out no matter if it's consumed locally on disk from a `file` location, or as uploaded on a VCS.

### `spec.target` [optional]
- A single target as a string. Can be either an absolute path/URL (depending on the type), or a relative path such as `./details/catalog-info.yaml` which is resolved relative to the location of this Location entity itself.

### `spec.targets` [optional]
- A list of targets as strings. They can all be either absolute paths/URLs (depending on the type), or relative paths such as `./details/catalog-info.yaml` which are resolved relative to the location of this Location entity itself.

### `spec.presence` [optional]
- Describes whether the target of a location is `required` to exist or not. It defaults to 'required' if not specified, can also be `'optional'`.

# acknowledgment
## Contributors

APA 🖖🏻

## Links
- Well known Annotations --> https://backstage.io/docs/features/software-catalog/well-known-annotations
- Well known Relations --> https://backstage.io/docs/features/software-catalog/well-known-relations


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
