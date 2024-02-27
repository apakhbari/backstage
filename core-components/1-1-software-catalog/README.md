# 1-1 Core Features: software catalog
```
 _______  _______  _______  _______  _     _  _______  ______    _______        _______  _______  _______  _______  ___      _______  _______ 
|       ||       ||       ||       || | _ | ||   _   ||    _ |  |       |      |       ||   _   ||       ||   _   ||   |    |       ||       |
|  _____||   _   ||    ___||_     _|| || || ||  |_|  ||   | ||  |    ___|      |       ||  |_|  ||_     _||  |_|  ||   |    |   _   ||    ___|
| |_____ |  | |  ||   |___   |   |  |       ||       ||   |_||_ |   |___       |       ||       |  |   |  |       ||   |    |  | |  ||   | __ 
|_____  ||  |_|  ||    ___|  |   |  |       ||       ||    __  ||    ___|      |      _||       |  |   |  |       ||   |___ |  |_|  ||   ||  |
 _____| ||       ||   |      |   |  |   _   ||   _   ||   |  | ||   |___       |     |_ |   _   |  |   |  |   _   ||       ||       ||   |_| |
|_______||_______||___|      |___|  |__| |__||__| |__||___|  |_||_______|      |_______||__| |__|  |___|  |__| |__||_______||_______||_______|
```

## Index



## Overview
- The Backstage Software Catalog is a centralized system that keeps track of ownership and metadata for all the software in your ecosystem (services, websites, libraries, data pipelines, etc). The catalog is built around the concept of metadata YAML files stored together with the code, which are then harvested and visualized in Backstage.
- Software Catalog enables two main use-cases:
1. Helping teams manage and maintain the software they own. Teams get a uniform view of all their software; services, libraries, websites, ML models — you name it, Backstage knows all about it.
2. Makes all the software in your company, and who owns it, discoverable. No more orphan software hiding in the dark corners of your software ecosystem.

### Adding components to the catalog
- The source of truth for the components in your software catalog are metadata YAML files stored in source control (GitHub, GitHub Enterprise, GitLab, ...).
- There are 3 ways to add components to the catalog:
1. Manually register components
2. Creating new components through Backstage
3. Integrating with an external source 

#### 1. Manually register components
- Users can register new components by going to /create and clicking the REGISTER EXISTING COMPONENT button. Backstage expects the full URL to the YAML in your source control. It is important to note that any kind of software can be registered in Backstage. Even if the software is not maintained by your company (SaaS offering, for example) it is still useful to create components for tracking ownership.

#### 2. Creating new components through Backstage
- All software created through the Backstage Software Templates are automatically registered in the catalog.

#### 3. Static catalog configuration
In addition to manually registering components, it is also possible to register components through **static configuration**. For example, the above example can be added using the following configuration:‍
```
catalog:
  locations:
    - type: url
      target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/components/artist-lookup-component.yaml
```

### Updating component metadata
- Teams owning the components are responsible for maintaining the metadata about them, and do so using their normal Git workflow.
- Once the change has been merged, Backstage will automatically show the updated metadata in the software catalog after a short while.

### Finding software in the catalog
- By default the software catalog shows components owned by the team of the logged in user. But you can also switch to All to see all the components across your company's software ecosystem. Basic inline search and column filtering makes it easy to browse a big set of components.

### Starring components
- For easy and quick access to components you visit frequently, Backstage supports starring of components:

### Integrated tooling through plugins
- The software catalog is a great way to organize the infrastructure tools you use to manage the software. This is how Backstage creates one developer portal for all your tools. Rather than asking teams to jump between different infrastructure UIs (and incurring additional cognitive overhead each time they make a context switch), most of these tools can be organized around the entities in the catalog.

## The Life of an Entity
### Key Concepts
- The catalog forms a hub of sorts, where entities are ingested from various authoritative sources and held in a database, subject to automated processing, and then presented through an API for quick and easy access by Backstage and others. The most common source is YAML files on a standard format, living in version control systems near the source code of systems that they describe. Those files are registered with the catalog and maintained by the respective owners. The catalog makes sure to keep itself up to date with changes to those files.

- The main extension points where developers can customize the catalog are:
1. **Entity providers**, that feed initial raw entity data into the catalog,
2. **Policies**, that establish baseline rules about the shape of entities,
3. **Processors**, that validate, analyze, and mutate the raw entity data into its final form.

- The high level processes involved are:
1. **Ingestion**, where entity providers fetch raw entity data from external sources and seed it into the database,
2. **Processing**, where the policies and processors continually treat the ingested data and may emit both other raw entities (that are also subject to processing), errors, relations to other entities, etc.,
3. **Stitching**, where all of the data emitted by various processors are assembled together into the final output entity.
An entity is not visible to the outside world (through the catalog API), until it has passed through the last process and landed among the final entities.

-An entity is not visible to the outside world (through the catalog API), until it has passed through the last process and landed among the final entities.

IMG
![life-of-an-entity_overview.svg](./images/life-of-an-entity_overview.svg)

#### Ingestion
- Each catalog deployment has a number of entity providers installed. They are responsible for fetching data from external authoritative sources in any way that they see fit, translating those into entity objects, and notifying the database when those entities are added or removed. These are the **unprocessed** entities that will be subject to later processing (see below), and they form the very basis of existence for entities. If there were no entity providers, no entities would ever enter the system.

- The database always keeps track of the set of entities that belong to each provider; no two providers can try to output the same entity. And when a provider signals the removal of an entity, then that leads to an **eager deletion**: the entity and all auxiliary data that it has led to in the database is immediately purged.

IMG
![life-of-an-entity_ingestion.svg](./images/life-of-an-entity_ingestion.svg)

- **2 default providers**:
1. The one that deals with user registered locations (e.g. URLs to YAML files)
2. The one that deals with static locations in the app-config.

- You can add more third party providers by passing them to the catalog builder in your backend initialization code, and you can easily write your own.
- An entity provider is a class that implements the `EntityProvider` interface. It has three main parts:
1. The identity: Each provider instance has a unique, stable identifier that the database can use to keep track of the originator of each unprocessed entity.
2. The connection: During backend startup, each provider is attached to the catalog runtime.
3. The stream of events: During its lifetime, the provider can issue change events to the runtime at any point in time, to modify its set of unprocessed entities.

- It is entirely up to the provider to choose how and when it produces these change events. For example, the app-config provider only fires off an update at startup and then lies dormant. The location database provider does an initial update at startup, and then small delta updates every time a location database change is detected. The LDAP provider is driven externally by a timer loop that occasionally triggers a full update. Some future provider may be entirely event driven, feeding off an event bus or web hook. There is no magic coordination among providers; if they need to arrange synchronization or locking among themselves for example to avoid duplicate work across multiple catalog service machines, they need to handle that out-of-band.

- The entities that are emitted get some coarse validation applied to them, to ensure that they at least adhere to the most basic schema rules about how an entity should be shaped. For example, they need to have a kind, a metadata.name, and optionally a metadata.namespace, among others. Apart from that, the ingestion stage considers its work done, and stores the unprocessed entities to be picked up at a later time by the processing system. This means that the more precise validation rules that you put in place on entities are _not yet_ applied at this stage.

#### Processing
- Every unprocessed entity comes with a timestamp, which tells at what time that the processing loop should next try to process it. When the entity first appears, this timestamp is set to "now" - asking for it to be picked up as soon as possible.

- Each catalog deployment has a number of processors installed. They are responsible for receiving unprocessed entities that the catalog decided are due for processing, and then running that data through a number of processing stages, mutating the entity and emitting auxiliary data about it. When all of that is done, the catalog takes all of that information and stores it as the processed entity, and errors and relations to other entities separately. Then, the catalog checks to see what entities are touched by that output, and triggers the final assembly of those (see Stitching below).

IMG
![life-of-an-entity_processing.svg](./images/life-of-an-entity_processing.svg)

- Entities are always processed one by one, but all of your catalog service hosts collaborate in doing so to distribute the load. Note how each processor can contribute to one or more of the fixed steps in the processing pipeline. First all of the processors' contributions to one step are run in the order that the processors were registered, then all of their contributions to the next step in the same order, and so on.

- Each step has the opportunity to optionally modify the entity, and to optionally emit other information. For example, the processor might look at information in the spec field of the entity, and emit relations that correspond to those declarations. If the processor emits an entity, then that entity gets stored verbatim with a timestamp saying that it, too, should be processed as soon as possible. If errors are emitted, then that signals that something is wrong with the entity and that it should not replace whatever previously error-free version we had among the final entities. If relations are emitted, then they are put in a dedicated relations table to be picked up by the stitching process below.

> Optional low level detail note: When entities are emitted, the catalog keeps track of the edges between the emitting entity and the ones emitted. This happens behind the scenes, hidden from the outside, and is used to form a graph. This is not the same thing as relations! The purpose of these edges, is to be able to detect when an entity becomes orphaned (see below), and to be able to perform eager deletions throughout the graph when a root is explicitly unregistered and nothing else is keeping lower nodes alive. We will talk more about orphaning and deletions later on in this article.

- When the final step has completed, and no errors were encountered, the processed entity and all of the relations are finally persisted in the database. Then the catalog considers this entity, and all of the entities it had relations to, subject for stitching.

- It is worth noting here that the processing does not lead to deletion or unregistration of entities; it can only call new entities into existence or update entities that it has previously called into existence. More about that later.


#### Stitching
- IMPORTANT: The stitching is currently a fixed process, that cannot be modified or extended. This means that any modifications you want to make on the final result, has to happen during ingestion or processing.

- Stitching finalizes the entity, by gathering all of the output from the previous steps and merging them into the final object which is what is visible from the catalog API. As the final entity itself gets updated, the stitcher makes sure that the search table gets refreshed accordingly as well.

> Note: The search table mentioned here is not related to the core Search feature of Backstage. It's rather the table that backs the ability to filter catalog API query results.

IMG
![life-of-an-entity_stitching.svg](./images/life-of-an-entity_stitching.svg)

- The diagram shows how the stitcher reads from several sources:
  - The processed entity, as returned from the processing step
  - The errors, if any, that were emitted by the processing step
  - All relations that were emitted by the processing step, as well as any relations emitted by other entity processing steps that happen to point at the current entity
 
- The last part is noteworthy: This is how the stitcher is able to collect all of the relation edges, both incoming and outgoing, no matter who produced them.

#### Errors
- Errors during the ingestion and processing of entities can happen in a variety of ways, and they may happen at a far later point in time than when they were registered. For example, a registered file may get deleted in the remote system, or the user may accidentally change the file contents in such a way that they cannot be parsed successfully, etc.

- There are two main ways that these errors are surfaced.

1. The catalog backend will produce detailed logs that should contain sufficient information for a reader to find the causes for errors. Since these logs are typically not easily found by end users, this can mainly be a useful tool for Backstage operators who want to debug problems either with statically registered entities that are under their control, or to help end users find problems.
2. For most classes of errors, the entity itself will contain a status field that describes the problem. The contents of this field is shown at the top of your entity page in Backstage, if you have placed the corresponding error callout component (`EntityProcessingErrorsPanel`) there.

#### Orphaning
- As mentioned earlier, entities internally form a graph. The edges go from processed parent entities, to child entities emitted while processing the parent.

- The processing loop runs continuously, so these edges are reconsidered over time. If processing a parent entity no longer emits a given child entity, then that former edge is severed. If that child has no other edges pointing at it either, it becomes orphaned. The end result is as follows:

- The stitching process injects a `backstage.io/orphan: 'true'` annotation on the child entity.
- The child entity is not removed from the catalog, but stays around until explicitly deleted via the catalog API, implicitly if `orphanStrategy: delete` configuration is set, or until it is "reclaimed" by the original parent or another parent starting to reference it.
- The catalog page in Backstage for the child entity detects the new annotation and informs users about the orphan status.


Orphaning can occur in several different scenarios:
- If a catalog-info YAML file is moved from one place to another in the version control system without updating the registration in the catalog, it will effectively become orphaned "by" that registered location
- If the user edits a corresponding parent catalog-info YAML file removing the entity's entry - for example in the case of a Location parent entity, orphaning can happen if editing or removing the target/targets lines pointing to the file containing the child entity.
- Another common cause is large batch processors such as the ones that crawl through remote systems looking for entities, no longer finding something that it used to find before. Maybe the data was moved, or deleted, in the remote system. So for example when a person leaves the company an LDAP org discovery processor might leave an orphaned User entity behind. Note that this only applies to processors - ingestion that happens using entity providers work differently, described below.

> Note that removing a file, or accidentally corrupting a file so that it cannot be read successfully, does not lead to orphaning. Hard errors, including the inability to find or read a distinct remote, are marked as such on the entity to inform the owner that something is wrong. But processing and other behaviors continue as usual.

The reason that the orphaning mechanism exists instead of having an eager deletion triggered, is safety. Scenarios like these can happen purely by accident, due to the asynchronous nature of the system and the fallible nature of humans. In particular when external systems start consuming and relying on the catalog, there could be substantial consequences to suddenly dropping entities without explicit owner consent. The catalog therefore takes the stance that entities that often were added by direct user action should also be deleted only by direct user action.

However, if you want to delete orphaned entities automatically anyway, you can enable the automated clean up with the following app-config option.
```
catalog:
  orphanStrategy: delete
```

#### Implicit Deletion

#### Explicit Deletion

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
