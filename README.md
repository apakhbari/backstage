# backstage
```
 _______  _______  _______  ___   _  _______  _______  _______  _______  _______ 
|  _    ||   _   ||       ||   | | ||       ||       ||   _   ||       ||       |
| |_|   ||  |_|  ||       ||   |_| ||  _____||_     _||  |_|  ||    ___||    ___|
|       ||       ||       ||      _|| |_____   |   |  |       ||   | __ |   |___ 
|  _   | |       ||      _||     |_ |_____  |  |   |  |       ||   ||  ||    ___|
| |_|   ||   _   ||     |_ |    _  | _____| |  |   |  |   _   ||   |_| ||   |___ 
|_______||__| |__||_______||___| |_||_______|  |___|  |__| |__||_______||_______|
```

## Index
1. Tips & Tricks
2. System Architecture
3. Components
4. Structure
5. Backstage Threat Model
6. Authentication
7. For Developing Plugins
8. To Be Organized Later
    - cache
    - Containerization
    - Plugins
    - Release & Versioning Policy

## Tips & Tricks
-  The sign-in system in Backstage does not exist to limit access, only to inform the system of the identity of the user.


## System Architecture
1. Back End
2. Front End
3. DB

- BE & FE can be in a single container but it is for dev purposes and not recommended.
- DB is postgres, but can use SQLite for dev purposes, if you use SQLIte it is going to be ephemeral so every time you restart it, data will lose

## Components
- Backstage’s core is composed of around 25 packages, which include a CLI, utility tools, API definitions, themes, and helpers. But what really makes up Backstage are the more than 150 open source plugin packages available, which include the framework’s main features.

### 1- Core
Deployed by default (e.g. software catalog)
1. Software catalog: all info about system & organization
2. Service Template: bootstrap services in backstage
3. Tech Docs: Documentatin inside backstage
4. Search: search across all backstage portal
5. Kubernetes cluster visualizer

### 2- App (Integrations)
Extend capabilities of backstage. Deployed but require configuration (e.g. SSO / Analytics / Git)

### 3- Plugins
Require installation, integration & configuration to your backstage application manually

Types of plugins:
- Service backed Plugins: Service is in same cluster/entity. deployed on different service in same system
- proxy backed plugin: redirect to proxy, which fetch info from some service outside of this cluster/entity

In Depth:
### 1-1 Core: software catalog
#### software catalog - Intro
- The Catalog’s objective is to map all software assets in your organization, including websites, APIs, libraries, and resources, in a centralized directory. This centralization is aimed at helping teams manage technology and enable discoverability. The Catalog tracks each asset's metadata, ownership, and dependencies, resulting in a software graph that surfaces orphaned entities.
- The Catalog is flexible enough to host a wide variety of software assets, known as entities in Backstage. Because a website is very different from a data processing pipeline, entities can be differentiated by kind. Moreover, even within kinds of entities, you can define types.
- The Catalog is powered by metadata stored in YAML files, which describe the kind, type, name, owner, and more details of a single entity per file. These files are commonly stored along their respective codebase, such that it gets updated frequently. Some entity kinds may have authoritative sources that are not represented in a codebase, for example, users and teams dictated by Okta.
- Tracking ownership and dependencies is one of the strongest use cases for the Catalog. These are also declared in the YAML file describing a software component. Only a single team can be the component owner, and this team must be registered in the Catalog as an entity as well. As for dependencies, Backstage lets you define how the component depends on another entity and what it exposes so others can consume it.

#### software catalog - In Depth
Registering Components in the Catalog
- The most common kind of entity that you’ll handle in your Backstage instance are components. Components refer to software components such as services, websites, and libraries that are typically linked to a repository whose code produces deployed instances or linkable artifacts. In Backstage, components are described with metadata in a YAML file that is stored in the repository where their code lives.
- Backstage needs to know where those YAML files are located in order to add them to the Catalog.
- There are two ways to do this:
1. manually registering a component through Backstage’s UI
2. setting up an entity processor that discovers YAML files in your organization’s source code management system.

Second way of registering components:
- Once you register these locations in the Catalog, Backstage will fetch the YAML files from them. Therefore, Backstage needs access to read your repository and will need an access token. Backstage will periodically check the information in the metadata files to keep the Catalog up to date. (set up an integration with GitHub)

Describing a Component in a YAML File
- Let’s say your organization has a public-facing website that you want to register in the Catalog. A description of that website could look something like this:
```
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: terroir-tracking-web
  description: Find where your juice comes from
spec:
  type: website
  lifecycle: production
  owner: tracking-team
```
- In the snippet above, notice that **apiVersion**, **kind**, and **metadata.name** are mandatory for all entities. And for a component, such as a website from the example, you must also specify type, lifecycle, and owner in spec.
- By default, Backstage recognizes a component to be any type from the following: website, service, and library. As for lifecycle, the framework recognizes production, experimental, and deprecated. You can use these attributes to customize how components are rendered in your Catalog.

- Once you have a YAML file to describe your component, you’ll need to add it to the component’s repository and make it available on its default branch. The rationale behind this is keeping the component’s metadata close to its source code, so maintainers can more easily keep the information up-to-date.
Although you can name your YAML file as you wish, Backstage recommends naming it catalog-info.yaml for uniformity.

Registering a GitHub Integration
- Integrations in Backstage lets you read or publish data using external sources such as GitHub, GitLab, Bitbucket, and other Cloud providers. Backstage offers 10 different vendor integrations by default. To register components in the Catalog, you must set up an integration so Backstage can fetch the YAML files from your repositories. Let’s get started by setting up a simple integration for GitHub.
- Consider using a GitHub App to manage the integration when setting up Backstage for your organization. A GitHub app gives you higher rate limits and lets a team manage the integration rather than a singular person. But for this course’s purposes, let’s keep it simple by using a Personal Access Token.

Manually registering components:
- the work can be tedious if your organization has dozens or hundreds of repositories.
- For surfacing component locations automatically from your organization, Backstage offers entity providers for GitHub, Gitlab, Azure DevOps, and Bitbucket. The entity provider will scan your organization’s repositories looking for metadata files to register them in the Catalog. You can configure how frequently Backstage will scan your organization in each service.
- When setting up an entity provider, a common issue that Backstage adopters run into is rate limits from the source code control system’s APIs. But you can leverage using different types of tokens to alleviate this issue; for example, a GitHub App token has a higher rate limit than a personal access token.

Understanding the Lifecycle of an Entity
- By registering a YAML file location, you’ve already added your first component to the Catalog. Under the hood, Backstage used an entity provider to store it as a raw entity in the database, then ran it through a series of processors and stitched together the processors’ outcomes into a final entity. Only after all this is done you could see the entity page in your Catalog. 

IMG
![Entity lifecycle.png](./images/overview_entity-lifecycle.png)

- All entities in Backstage come from entity providers, which take data from authoritative sources. Entity providers import raw data from their source and store it in a database in a process called ingestion. A Backstage instance typically uses more than one entity provider, for example, GitHub for repositories, Okta for users, and AWS S3 for resources. Entity providers can issue updates on their associated entities when they see fit and determine when raw entities are to be processed. 
- When marked for processing, a raw entity goes through a series of steps with processors that may change their data format, extract relationships, emit errors, or even create new entities. Entities are processed one by one but can involve all of your services. You can add and configure processors in your instance as you want.
- After processing is done, the information resulting from processing steps that impact the current entity is stitched together into a final entity that can be consumed through the Catalog API, which is used to generate pages and more in your Backstage instance. The stitching process is fixed and doesn’t allow any customizations, so you need to make sure you transform the data in the ingestion or processing stages.

Understanding Locations
- locations are internal to Backstage, knowing what they are might be useful when setting up the Catalog.
- Location is a kind of entity in Backstage. It is used for keeping track of where Catalog information can be found. In this case, the generated location targets the YAML file repository.
- All entities registered in the Catalog rely on locations, not only those linked to repositories like components. Each entity provider is responsible for determining the canonical URL used as the location for an entity. Furthermore, a location may have more than one target. For example, when registering components that live in a monorepo, a single location may reference all the services stored there. 
- You normally do not have to interact with location entities because Backstage generates them. You won’t even see them in the dependency graph for your components, although all of them are associated with a location. But given that they represent the contact point between your instance and external sources when something goes wrong, you might need to check if the locations are working correctly.

Orphaned Entities and Deletions
- The framework continuously evaluates entities through the process outlined on the previous page. Thus, it updates the dependency graph in the Catalog over time. When a parent entity no longer emits a child entity, the child entity is labeled as an orphan. Whether an entity is orphaned is important when deleting an entity. 
- Orphaned entities occur when a parent entity no longer emits a child entity, severing the relationship which existed between them before. When this is detected, the stitching process injects a backstage.io/orphan: 'true’ annotation on the child entity but doesn’t remove it from the Catalog. The orphaned entity can then be deleted or reclaimed by another parent entity.
- Orphanining can occur when a catalog-info file is moved in the repository without updating the Catalog registration or when a user removes the entity’s entry from the parent YAML file. Orphaning may also occur in batch processing when the crawler doesn’t find a reference where it expected it to be.
- Whether an entity is orphaned may be a condition to delete an entity successfully. There are two kinds of deletion in Backstage: implicit deletion and explicit deletion. When you mark an entity for deletion through the entity provider, you’re issuing an implicit deletion bound to other entities associated with it. If the entity has parents, you’ll be prompted to issue the deletion through a parent. If it’s orphaned, the entity will be deleted as part of the processing loop.
- Explicit deletion can be issued through the Catalog API. It makes sense to use this approach with abandoned entities or entities that are not being kept up to date by any other parent entity. The reason is that if you delete an entity through explicit deletion, it’ll re-appear in the Catalog if a parent entity references it again.

Defining a Taxonomy for Your Entities
- You can register different kinds of entities in Backstage and different types for each kind.  By default, Backstage recognizes nine kinds, with pre-defined types in some cases. However, you can better define the kinds and types that represent your organization.

- Let’s review the basic kinds that you’ll work with most of the time: 
1. Component refers to a software component bound to a repository and is deployable or linkable in an Artifactory. There are three known types of components: service, website, and library.
2. API refers to an interface that can be exposed by a component. Backstage ships with support for OpenAPI, AsyncAPI, GraphQL, and gRPC, but it doesn’t have an opinion on the types you may use.
3. Resource refers to infrastructures like databases, storages, or CDNs. Bringing resources to your Catalog and associating them with components can be useful to visualize and optimize your system.
4. Group is used to arrange users into teams or business units. Backstage lets you model your organization hierarchy using groups that belong to other groups.
5. User refers to a person, such as an employee or contractor. Backstage requires you to assign all users to a group.

- Backstage uses kinds and types to organize entities and customize how they are presented in the Catalog. For example, you can make your Catalog show a visitors analytics dashboard in an entity of kind component and type website but show an incidents report for a component with type service instead.
- Backstage allows you to set arbitrary text in the kind and type fields of any catalog-info file, but it won’t do anything special with that information until you define behavior in your instance. To start using custom kinds, you need to configure your catalog rules to pick up your kind, as it only recognized the default kinds. For custom types, you’ll need to decide what to do with the type, for instance, showing a certain integration on its entity page.


Modeling Your System in Backstage
- Backstage intends to gather all your software assets in one place. However, bringing them all together in a long list would make it harder for your users to navigate through. Just as you can map your organization hierarchy with Group and User entities, you can model your ecosystem using System and Domain entities. 
- **System** is a default kind in Backstage whose purpose is to abstract away details of a unit, letting the user know what the system has to offer without getting into how it is composed. As such, a system can be comprised of several components and resources, but will only expose the APIs contained within it and those it depends on. 
- **Domain** is also a default kind in Backstage, meant to arrange together systems and documentation regarding a business unit or other kind of bounded context. The definition is quite loose so you can aggregate together what makes the most sense for your case.  
- Domain and systems will be visible in the “Explore” tab of your Backstage instance. From there, users can more easily find relevant information without having to know the name of the component or service they may need.

Relationships
- Entities can be related to each other in more than one way. Relationships in Backstage are read-only and directional. This means you define how the source entity is related to a target entity. All relationships are mapped into the software graph, which can be visualized on your Catalog entity’s page.

- Let’s review the built-in relationship types that your entities can use:
1. **providesApi** states a component or system provides an API for consumption. apiProvidedBy signifies the other direction of this relationship.
2. **consumesApi** expresses a component or system consumes the API provided by another entity.  **apiConsumedBy** signifies the other direction of this relationship.
3. **dependsOn** means a component depends on another entity, for example a website component that is built using a library component. **dependencyOf** signifies the other direction of this relationship.
4. **parentOf** and **childOf** are used to build trees with entities, for example when modeling an organization with groups. This relationship is commonly defined in **spec.parent** and/or **spec.children**.
5. **memberOf** and **hasMember** defines a membership relationship, used for users and groups.
6. **partOf** states a component or system is part of a larger component, a system, or domain. This relation is commonly defined in **spec.system** or **spec.domain**.

Ownership
- In Backstage, ownership is meant to answer who’s ultimately responsible for each component, API, system, or domain. There can only be one owner for an entity, preferably a group rather than a user. The owner is expected to be able to maintain the metadata and be the contact point for the entity.
- Ownership is typically defined in **spec.owner** of the owned entity, which describes the relationships **ownedBy**. **spec.owner** is a required field for all components, APIs, resources, systems, and domains you register in the Catalog. Each group or user will get a corresponding **ownerOf** relationship for each entity they own. 
- If you already manage ownership in your repositories through CODEOWNERS, you can let Backstage use this reference instead of filling in the **spec.owner** field for the applicable entities. This feature is available through the CodeOwnersProcessor module of the Catalog.

### 1-2 Core: Scaffolder
#### Scaffolder - Intro
- The Scaffolder provides your developers with the ability to execute software templates that initialize repositories with skeleton code and predefined settings.
- The Scaffolder is a perfect place for new engineers to jump into the development process right away. You could, for example, set up a template to Create Node/React Website in your Scaffolder, which sets up a repository with CI/CD and analytics baked in from the beginning. When the new developers use the software template, they’ll get a deploy-ready service that will allow them to familiarize themselves with the tools and feel productive with a few clicks instead of having to wander aimlessly through your tech ecosystem. 
- A software template is defined in a YAML file that specifies parameters and steps to execute. Backstage will generate a UI in the Scaffolder based on the parameters that you specify in your software template. For the steps, you can leverage built-in actions for common fetch operations, but you can also define your own. 
- Templates are stored in the Catalog under a template kind. Furthermore, all components initialized by the Scaffolder are automatically added to the Catalog. Therefore, there’s a virtuous cycle between the Scaffolder and the Catalog that promotes discoverability and standardization.


#### Scaffolder - In Depth
- The Scaffolder enables you to provide software templates to your teams. Software templates are composed of a YAML definition and, typically, a skeleton directory. You can define the parameters that will be asked from the user and the steps that the template will execute. The community templates can help you get ideas of the templates' potential.
- Software templates are defined using YAML files and are registered in Backstage the same way other entities are. Software templates are defined by setting the entity kind to template, which requires parameters and steps to be declared.
- Let’s review a Hello world template to illustrate how templates are defined.

```
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: hello-world-template
  title: Hello World
  description: Says Hello to a specified name.
spec:
  owner: terroir-ops
  type: service

  parameters:
    - title: You are about to say hello to your first Backstage Template
      required:
        - name
      properties:
        name:
          type: string

  steps:
    - id: log-message
      name: Log Message
      action: debug:log
      input:
        message: 'Hello, ${{ parameters.name }}!'
```
- As with other entities, **apiVersion**, **kind**, and **metadata.name** are required fields.  **spec.owner** and **spec.type** are also required, where the owner is the team responsible for this template, and type is the resulting component type: if the template will initialize a website, then the template is expected to have a type website.
- With each parameter you define, the Scaffolder will generate an input in the template form.
- As for steps, you can define what happens in each of them. A few actions are available by default, and you can define your own too. You can use the parameters inputted by the user in the steps.
- IMPORTANT: To be able to preview templates in UI, you need to use a browser with support for the File System Access API, such as Chrome.

Defining Parameters in a Software Template
- In a software template, the parameters property is used to generate a form, with one or multiple form steps, that the user needs to fill in when executing a template. In each form step, you can define properties, which will be mapped to inputs to ask for information from the user.
- The parameters property accepts an array of objects. Each entry in the array is a form step. Form steps are not bound to the parameter step from the template definition. Thus, you can break up your form into the steps that make the most sense to your users and use any of those values across any template step later on.
- Each form step must specify a title, which properties it will ask from the user, and if any of them is required. While defining your parameters, the Template Editor > Edit Template Form tool is useful for previewing how your form will look like.
- Let’s define the parameters for a two-form steps template, where you ask for a simple string from the user in each form step.  The parameters would look like this:

```
parameters:
  - title: First form step
    required:
        - text-a
    properties:
      text-a:
        title: Some text A
        type: string
  - title: Second form step
    required:
      - text-b
    properties:
      text-b:
        title: Some text A
        type: string
steps: 
  - id: fetch-base
    name: Fetch Base
    action: fetch:template   
input:
      url: ./template
      values:
        name: ${{parameters.name}}
```

- Now, let’s talk about the properties that you define in each form step. The most basic property type is a string, as the one that can be seen in the example for text-a and text-b. However, you can customize the input rendered to obtain this string by declaring a ui:field. 
- Backstage comes with built-in ui:fields that make it easier for the user to input a valid value. For instance, EntityPicker will let the user select from the entities registered in the Catalog, as pictured below:
- Other examples for ui:field are OwnedEntityPicker, which shows a list of entities owned by the user, EntityNamePicker which validates the user inputs a valid entity name, and RepoUrlPicker, which accepts criteria of repositories to be shown as options. 
- Other than strings, parameters can also be of type number, array, and object.  You can also specify options for each parameter, such as default values or a fixed list of possible values. 
- Once you have defined your form steps and parameters, you can use them in your steps using the parameter key and the name of the parameter. For example:
```
parameters:
  properties:
    name:
      type: string
steps:
  - id: log-message
    name: Log Message
    action: debug:log
    input:
      message: 'Hello, ${{ parameters.name }}!'
```

Defining Steps in a Software Template
- Steps define the actions that are taken by the software template when it is run. The Scaffolder initially creates a temporary directory referred to as the workspace, in which files are downloaded, generated, updated, and pushed to some external system. Steps are executed in consecutive order according to the template definition.
- Backstage ships with some useful actions, and you can define your own too. To check which actions are available in your instance and how to use them, go to the Create page, click on the top right context menu and then click on “Installed Actions”. On that page, you’ll find documentation for the actions that you can use in your steps and how to use them.

### 1-3 Core: TechDocs
#### Scaffolder - Intro
- a centralized hub for all their documentation
- TechDocs is the framework’s documentation-as-code solution; it takes markdown files and transforms them into static pages
- TechDocs follows the same principle as the Catalog metadata files: stay close to the source code to stay accurate. TechDocs are written as markdown files in the repository where the entity that they document is kept. Then, the TechDocs Backstage plugin fetches these files from all services, generates static pages, and publishes them.

#### Scaffolder - In Depth
- he TechDocs plugin lets you add documentation for your software components directly into their corresponding entity page in Backstage. TechDocs is based on markdown files stored alongside the code that they document to make it easier for developers to keep it up to date. TechDocs allows you to define navigation and pages, which will be made available under the “Docs” tab of its associated component.
- Docs are associated with a component when their YAML file includes the backstage.io/techdocs-ref annotation. The annotation tells Backstage that there are docs available that it should show to the user.

Build Strategies in TechDocs
- TechDocs goes through three stages to make documentation from markdown files in a repo to pages deployed in your Backstage instance:
1. preparation
2. generation
3. publishing

- During the preparation stage, TechDocs clones the component’s repository into a temporary directory to be able to access the docs locally. Then, the generation stage proceeds by leveraging a docker image called mkdocs-techdocs-core, which uses a Python library to build the static assets that compose the documentation. At last, in the publishing stage, the assets are moved somewhere so they can be deployed.
- Although the process described above is simple to get started with, it comes with limitations when using it in production. For example, building the documentation uses a Docker image, which makes it difficult to run in, for example, a Kubernetes pod. You could use the image binaries, but then you’d be adding non-core Backstage dependencies in your Dockerfile. 
- Furthermore, building the documentation on the fly will result in slow first requests and duplicated work if you have multiple backends configured in your instance, as TechDocs will generate the docs for each of them. Additionally, pulling down the entire source code of a component to generate docs might not meet your security expectations. 
- For these reasons, you can tell TechDocs that you will opt out of its built-in generation—by specifying techdocs.builder to external in the TechDocs annotation—and publish the generated assets in a Cloud storage. Typically, you’d rely on your CI system to generate documentation when a Pull Request is merged. The generated assets will then be published in a Cloud storage, such as an S3 Bucket, and your Backstage instance would read them when needed. 

Writing TechDocs
- TechDocs generates the documentation using MkDocs and styles it using Material for MkDocs, so there is nothing particularly “Backstage” about how you write your markdown files or configurations. 
- You can configure the navigation in the mkdocs.yml for your documentation, as you would normally do with MkDocs.
- For writing the documentation, you can use standard markdown. The Python library in charge of the generation complies with standard markdown syntax, with very minor differences. 
- For linking files within your docs, you can use standard relative markdown links.

### 1-4 Core: Search
-  the most recent addition to Backstage’s framework.
-  earch allows developers to find information across their ecosystem by leveraging your search engine of preference and lets you customize how things are indexed and presented to the user.
-  Search is quite customizable. For starters, the plugin allows you to bring your own search engine, although ElasticSearch is the officially maintained engine. Search ships with a rudimentary query translator that turns the user’s input into a fully formed query, but you’re allowed to customize it to your engine and use cases better. You’re also welcome to customize the search results page and what each result looks like.
-  Under the hood, Search searches “documents” that represent entities, documentation pages, or any other thing that you put into Backstage. These documents are consumed through streams exposed by a Collator. Collators define what can be found by defining, indexing, and collecting documents. Currently, collators are available for the Catalog, TechDocs, and Stack Overflow. You can define your collators too.

### 1-5 Core: k8s
- he Kubernetes plugin is tied to the Catalog. It shows information about the clusters associated with a service registered in the catalog. To enable it, you must tell Backstage how to discover your clusters, whether that is by reading information that exists already in the Catalog or by fetching it directly from GKE or another custom Kubernetes cluster supplier.

Categories of Plugins
1. Feature plugins: provides functionality to Backstage, such as the core plugins Catalog, Scaffolder, and TechDocs, and other plugins such as Org, Badges, and Tech Insights.
2. Extension plugins: extend a feature plugin with added functionality, for example, the gRPC module for API Docs or the Google Analytics module for the Analytics plugin.
3. Integration plugin: brings information from a vendor or external service into Backstage, typically in the form of a widget in the entity page or a standalone page. For example, the Argo CD plugin, GitHub Pull Request Board, or the CircleCI plugin.
4. Entity providers: bring entities into Backstage from their respective sources, for example, the Azure Entity Provider and the LDAP integration. 

### 3- Plugins
- Your Backstage app is a single-page application whose features you define with plugins. Backstage plugins are implemented as React frontends that you can plug into a Backstage app. The plugin system in Backstage is built around extensibility and composability. For this reason, Backstage provides special primitives and APIs that allow plugins to share data and provide extension points without getting into tangled implementations or dependencies. 

- Backstage plugins may also offer a backend counterpart that implements the API that the plugin frontend consumes. In some cases, teams opt out of a plugin’s frontend and only consume the plugin’s backend. This means they have to build and maintain their own frontend plugin but gain the ability to control exactly how they want their UI to work.

Categories of Plugin (not-official)
1. **Feature plugins:** provides functionality to Backstage, such as the core plugins Catalog, Scaffolder, and TechDocs, and other plugins such as Org, Badges, and Tech Insights.
2. **Extension plugins:** extend a feature plugin with added functionality, for example, the gRPC module for API Docs or the Google Analytics module for the Analytics plugin.
3. **Integration plugin:** brings information from a vendor or external service into Backstage, typically in the form of a widget in the entity page or a standalone page. For example, the Argo CD plugin, GitHub Pull Request Board, or the CircleCI plugin.
4. **Entity providers:** bring entities into Backstage from their respective sources, for example, the Azure Entity Provider and the LDAP integration. 

Installing Plugins in Backstage
- Depending on the category of the plugin you want to install, you’ll need to do certain steps in different places. To install a plugin, you’ll need to add their packages to your instance, and depending on the plugin, you might have to add annotations to your metadata files, authorize your instance to access a third-party service, or add the plugin’s UI into your instance.
- If you were to use the plugin’s provided UI, you’ll have to install all the packages. But if you want to provide your own UI, you can only use the plugin’s backend.
- Other than adding the plugin’s packages into your instance, feature plugins usually require you to set up information in the entities’ YAML files. For example, adding documentation annotations for TechDocs in each YAML file.
-  In the case of integration plugins, after you have added their packages to your instance, you’ll need to authorize Backstage to access the third-party service. The most common way to do this is via an authorization token for your instance to access the third-party API. 
- Finally, most plugins will require you to add their UI into your instance’s frontend using React. For example, adding a link in a navigation menu, or a widget on the entity page. Some plugins allow you to customize these widgets by passing props into them. 

## Structure
### Structure - Simple
When you navigate to the newly created directory, you’ll see the following structure:
```
─ packages
    ├─ app
            └─ package.json
    └─ backend
            └─ package.json
─ app-config.yaml
─ catalog-info.yaml
─ lerna.json
─ package.json
```
- Notice that there are several package.json, that’s because your Backstage instance is implemented as a monorepo. Thus, the frontend (app) and the backend (backend) have their own dependencies and can be deployed independently. To manage dependencies between the monorepo, Backstage uses lerna, hence the lerna.json file.
- Notice as well the catalog-info.yaml file. Backstage uses this metadata file to add itself to the Software Catalog. Yes, Backstage wants to track every software asset in your organization, including itself!
- At last, check out app-config.yaml. This is the main configuration file, where you can define your instance’s name and set options for the backend, authentication, and other integrations.
- You already worked on the main configuration file, app-config.yaml, to set your organization name. But you’ll find app-config.local.yaml and app-config.production.yaml too, these are used in local and production environments, respectively. 
- For now, you’ll be working with app-config.local.yaml. Backstage includes this file by default in .gitignore, which means nothing you put there will be committed nor pushed upstream. This can give you more peace of mind when putting secrets for local development there.
### Structure - In Depth
#### General purpose files and folders
In the project root, there are a set of files and folders which are not part of the project as such, and may or may not be familiar to someone looking through the code.

- **.changeset/** - This folder contains files outlining which changes occurred in the project since the last release. These files are added manually, but managed by changesets and will be removed at every new release. They are essentially building-blocks of a CHANGELOG.
- **.github/** - Standard GitHub folder. It contains - amongst other things - our workflow definitions and templates. Worth noting is the vale sub-folder which is used for a markdown spellchecker.
- **.yarn/** - Backstage ships with its own yarn implementation. This allows us to have better control over our yarn.lock file and hopefully avoid problems due to yarn versioning differences.
- **contrib/** - Collection of examples or resources contributed by the community. We really appreciate contributions in here and encourage them being kept up to date.
- **docs/** - This is where we keep all of our documentation Markdown files. These end up on https://backstage.io/docs. Just keep in mind that changes to the sidebars.json file may be needed as sections are added/removed.
- **.editorconfig** - A configuration file used by most common code editors. Learn more at EditorConfig.org.
- **.imgbotconfig** - Configuration for a bot which helps reduce image sizes.

#### Monorepo packages
```
 "workspaces": {
    "packages": [
      "packages/*",
      "plugins/*"
    ]
  },
```
Let's look at them individually.

#### packages/
These are all the packages that we use within the project. Plugins are separated out into their own folder, see further down.

- **app/** - This is our take on how an App could look like, bringing together a set of packages and plugins into a working Backstage App. This is not a published package, and the main goals are to provide a demo of what an App could look like and to enable local development.
- **backend/** - Every standalone Backstage project will have both an app and a backend package. The backend uses plugins to construct a working backend that the frontend (app) can use.
- **backend-common/** - There are no "core" packages in the backend. Instead we have backend-common which contains helper middleware and other utils.
- **catalog-client** - An isomorphic client to interact with the Software Catalog. Backend plugins can use the package directly. Frontend plugins can use the client by using @backstage/plugin-catalog in combination with useApi and the catalogApiRef.
- **catalog-model/** - You can consider this to be a library for working with the catalog of sorts. It contains the definition of an Entity, as well as validation and other logic related to it. This package can be used in both the frontend and the backend.
- **cli/** - One of the biggest packages in our project, the cli is used to build, serve, diff, create plugins and more. In the early days of this project, we started out with calling tools directly - such as eslint - through package.json. But as it was tricky to have a good development experience around that when we change named tooling, we opted for wrapping those in our own CLI. That way everything looks the same in package.json. Much like react-scripts.
- **cli-common/** - This package mainly handles path resolving. It is a separate package to reduce bugs in CLI. We also want as few dependencies as possible to reduce download time when running the CLI which is another reason this is a separate package.
- **config/** - The way we read configuration data. This package can take a bunch of config objects and merge them together. app-config.yaml is an example of an config object.
- **config-loader/** - This package is used to read config objects. It does not know how to merge, but only reads files and passes them on to the config. As this part is only used by the backend, we chose to separate config and config-loader into two different packages.
- **core-app-api/** - This package contains the core APIs that are used to wire together Backstage apps.
- **core-components/** - This package contains our visual React components, some of which you can find in plugin examples. (https://backstage.io/storybook/?path=/story/plugins-examples--plugin-with-data)
- **core-plugin-api/** - This package contains the core APIs that are used to build Backstage plugins.
- **create-app/** - An CLI to specifically scaffold a new Backstage App. It does so by using a template.
- **dev-utils/** - Helps you setup a plugin for isolated development so that it can be served separately.
- **e2e-test/** - Another CLI that can be run to try out what would happen if you build all the packages, publish them, create a new app, and then run them. CI uses this for e2e-tests.
- **integration/** - Common functionalities of integrations like GitHub, GitLab, etc.
- **storybook/** - This folder contains only the Storybook config which helps visualize our reusable React components. Stories are within the core package, and are published in the Backstage Storybook.
- **techdocs-node/** - Common node.js functionalities for TechDocs, to be shared between techdocs-backend plugin and techdocs-cli.
- **test-utils/** - This package contains general purpose testing facilities for testing a Backstage App or its plugins.
- **theme/** - Holds the Backstage Theme.

#### plugins/
- Most of the functionality of a Backstage App comes from plugins. Even core features can be plugins, take the catalog as an example.
- We can categorize plugins into three different types:
1. Frontend
2. Backend
3. GraphQL

- We differentiate these types of plugins when we name them, with a dash-suffix. -backend means it’s a backend plugin and so on.
- One reason for splitting a plugin is because of its dependencies. Another reason is for clear separation of concerns.
- Take a look at our Plugin Directory or browse through the plugins/ folder.

#### Packages outside of the monorepo
For convenience we include packages in our project that are not part of our monorepo setup.

microsite/ - This folder contains the source code for backstage.io. It is built with Docusaurus. This folder is not part of the monorepo due to dependency reasons. Look at the microsite README for instructions on how to run it locally.

## Backstage Threat Model
### Trust Model
- The Backstage trust model is divided into three groups with different trust levels.
- An **internal user** is an authenticated user that generally belongs to the organization of a particular Backstage deployment. These users are trusted to the extent that they are not expected to compromise the availability of Backstage, but they are not trusted to not compromise data confidentiality or integrity.
- An **integrator** is a user responsible for configuring and maintaining an instance of Backstage. Integrators are fully trusted, since they operate the system and database and therefore have root access to the host system. Additional measures can be taken by adopters of Backstage in order to restrict or observe the access of this group, but that falls outside of the current scope of Backstage.
- Another group of de facto integrators is internal and external code contributors. When installing Backstage plugins you should vet them just like any other package from an external source. While it’s possible to limit the impact of for example a supply chain attack by splitting the deployment into separate services with different plugins, the Backstage project itself does not aim to prevent these kinds of attacks or in any other way sandbox or limit the access of plugins.
- An **external user** is a user that does not belong to the other two groups, for example a malicious actor outside of the organization. The security model of Backstage currently assumes that this group does not have any direct access to Backstage, and it is the responsibility of each adopter of Backstage to make sure this is the case.

### Integrator Responsibilities
- As an integrator of Backstage you yourself are responsible for protecting your Backstage installation from external and unauthorized access. The sign-in system in Backstage does not exist to limit access, only to inform the system of the identity of the user. There are some plugins that have more fine-grained access control through the permissions system, but the primary purpose of that system is to restrict access to resources for internal users rather than Backstage as a whole. A common and recommended way to protect a Backstage deployment from unauthorized access is to deploy it behind an authenticating proxy such as AWS’s ALB, GCP’s IAP, or Cloudflare Access.
- Other responsibilities include protecting the integrity of configuration files as it may otherwise be possible to introduce vulnerable configurations, as well as the confidentiality of configured secrets related to Backstage as these typically include authentication details to third party systems.
- The integrator is also responsible for maintaining the resolved NPM dependencies of their Backstage project. This involves ensuring that yarn.lock receives updated versions of packages that have vulnerabilities, when those fixed versions are in range of what the Backstage packages request in their respective package.json files. This is commonly done by employing automated tooling such as Dependabot, Snyk, and/or Renovate on your own repository. When fixed versions exist that are not in range of what Backstage packages request, or when larger operations such as switching out an entire dependency for another one is required, maintainers collaborate with contributors to try to address those dependency declarations in the main project as soon as possible.

### Common Backend Configuration
- There are many common facilities that are configured centrally and available to all Backstage backend plugins. For example there is a DatabaseManager that provides access to a SQL database, TaskScheduler for scheduling long-running tasks, Logger as a general logging facility, and UrlReader for reading content from external sources. These are all configured either directly in code, or within the backend block of the static configuration. The appropriate care needs to be taken to ensure that any secrets remain confidential and no malicious configuration is injected.
- In a typical Backstage setup, there is no boundary between plugins that run on the same host. Likewise, there is no boundary between plugins that share the same database access. Any plugin that is running on a host that has access to the logical database of any other plugin should be considered to have full access to that other plugin. For example, even if you deploy the auth and catalog plugins on separate hosts with separate configuration and credentials, the catalog plugin is still considered to have full access to the auth plugin as long as the catalog plugin has access to the auth plugin's logical database. The only way to create a boundary between the two plugins is to deploy them in such a way that they do not have access to each others’ database. This applies to the database facility as well as any other shared resources, such as the cache.
- Note that the UrlReader system operates with a service context and is not integrated with the Backstage permission system or other external access control mechanisms. This means users of your Backstage instance may be able to read external content which that individual should not have access to. For example, the $text placeholder in a catalog-info.yaml can be used to read contents from a source such as a GitHub repository that the user does not have direct access too. If this is a concern it is recommended to either disable or replace the resolvers in the catalog placeholder processor and similar features in any other plugin.

### Authentication
- Backstage provides authentication of users through the auth plugin, which primarily acts as an authorization server for different OAuth 2.0 provider integrations. These integrations can both serve the purpose of signing users into Backstage, as well as providing delegated access to external resources, and are all subject to the common concerns of implementing secure OAuth 2.0 authorization servers. All auth provider integrations are disabled by default, and need to be enabled through configuration in order to be used. For each Backstage installation it is recommended to only enable the minimal set of providers that are in use by that instance.
- In order to use an auth provider to sign in users into Backstage, it needs to be configured with an Identity resolver, which is a custom callback implemented in code. The identity resolver is a sensitive part of configuring Backstage and it is important that it always resolves user identities correctly, based on information provided by the authentication provider. There are a number of built-in identity resolvers that can simplify configuration, and it is important that these all resolve users in a secure way, regardless of how they are used.
- As part of signing in with an identity resolver, a Backstage Token is issued containing the resolved user identity. The tokens are asymmetrically signed JSON Web Tokens, with the public keys available to any service that wishes to verify a token. The signing keys are rotated continuously and are unique to each installation of Backstage, meaning that Backstage Tokens are not shared across installations. The token contains claims for the user identity and ownership information, which can be used to determine what Backstage resources are owned by that user or group. It is important that this token can not be forged outside of the auth plugin, with the exception of other plugins deployed in the same backend service or sharing the same database. For a high-security deployment, the auth backend should therefore be deployed in a separate service with its own database.
- The token is used to prove the identity of the user within the Backstage system, and is used throughout Backstage plugins to control access. It is important that the ownership resolution logic is consistent across the entire Backstage ecosystem, with no possibility of misinterpreting the ownership information.
- For cross-backend communication, the Backstage Token is typically forwarded or, in strict backend-to-backend communication without a user party, the backend itself issues a service token based on a pre-shared secret which is then validated on the receiving end. There are no unique service identities tied to these tokens at this point, meaning the tokens can be used across all services in a Backstage installation. This is something that we aim to improve in the future.

### Catalog
- Integrators should configure catalog rules to limit the allowed entity kinds that users can define. In general it is best to restrict definition of User, Group, and Template entities so that internal users cannot register additional ones. Template entities define actions that are executed on the backend hosts, and while the goal is for these actions to be secure regardless of input, it is still a more sensitive context and it is recommended that you protect it with additional checks. It is very important to not allow registration of User and Group entities if you ingest and rely on these as organizational data in your catalog. Doing so could otherwise open up for the ability to impersonate users and confuse group membership information. You should always ingest organizational data using a statically configured catalog location or an entity provider reading from a trusted source. The entities emitted directly by an entity provider are always trusted and rules are not applied to them, but any entities produced further down the chain are still subject to the rules.
- By default all internal users are allowed to create and delete entities. If this does not fit your organization's needs it is recommended to enable and configure the permission system to restrict these operations.

### Scaffolder
- By default, Scaffolding jobs execute directly on the host machine, including any actions defined in the template. Because the Scaffolder templates are considered a more sensitive area it is recommended to control access to create and update templates to trusted parties. Template execution is intended to be secure regardless of input, but we still recommend this additional layer of protection. The string templating is executed in a node VM sandbox to mitigate the possibility of remote code execution attacks.
- The Scaffolder often has elevated permissions to for example create repositories in a Github organization. The integrator should therefore be cautious of Scaffolder Templates that for example delete or update existing resources as the user input is typically user defined and can therefore delete or modify resources maliciously or by mistake.
- One strategy that allows you to reduce the access that the Scaffolder service has is to rely on user credentials when executing actions. For example, a GitHub App integration could be configured with read-only permissions, with a separate user OAuth token used to create repositories. This requires that your users have access to create repositories in the first place.
- By default all internal users are allowed to execute templates in the scaffolder. If this does not fit your organization's needs it is recommended to enable and configure the permission system to restrict these operations.


### TechDocs
- TechDocs' backend can be broadly configured in two ways. The default is when techdocs.builder is set to local, in which documentation is generated on-demand and stored locally by the TechDocs backend. When techdocs.builder is set to external instead, documentation is assumed to be generated by an external process (e.g. in a CI/CD pipeline), and merely read from a configured external storage provider.
- When documentation is generated locally, integrators are responsible for ensuring secure configuration of file system permissions in the location where generated assets are stored. When documentation is generated externally, integrators are responsible for access control and permissioning between the external process that generates the documentation, the storage provider where documentation assets are published, and the TechDocs backend.

- Regardless of backend configuration, the TechDocs frontend does not trust the generated HTML of any documentation sites and therefore applies a strict sanitization process before rendering out any content to users.

- By default, all TechDocs documentation is visible to all Backstage users. Access can be restricted to TechDocs sites by configuring view permissions for the Catalog.

### Proxy
- The proxy backend acts as a utility for frontend plugins to access remote services that may not be set up to receive traffic directly from the Backstage frontend. Typical reasons for this would be that the upstream service does not provide the appropriate CORS headers or does not serve its content over HTTPS.

- The proxy entries are configured through static configuration. Each entry has a mount path and an upstream target, and also supports other options such as limiting the allowed methods, or injecting additional headers. It is recommended to avoid injecting authentication headers for upstream services, as this is a risky way to decorate requests with credentials. Anyone with access to your Backstage deployment will be able to make requests to the upstream service using the injected credentials. It is recommended that you instead create a backend plugin that forwards individual requests to the upstream service in a secure way. In case you do end up injecting credentials into upstream requests, be sure that you are not exposing any sensitive information or actions. You should also restrict the access as much as possible, for example using the allowedMethods option to limit the methods that can be used, and using tokens with the minimum required authorization scope.


## Authentication
- The authentication system in Backstage serves two distinct purposes: sign-in and identification of users, as well as delegating access to third-party resources. It is possible to configure Backstage to have any number of authentication providers, but only one of these will typically be used for sign-in, with the rest being used to provide access to external resources.

> NOTE: Identity management and the Sign-In page in Backstage is NOT a method for blocking access for unauthorized users. The identity system only serves to provide a personalized experience and access to a Backstage Identity Token, which can be passed to backend plugins. This also means that your Backstage backend APIs are by default unauthenticated. Thus, if your Backstage instance is exposed to the Internet, anyone can access information in the Backstage. You can learn more here.

### Sign-In Configuration
- Using an authentication provider for sign-in is something you need to configure both in the frontend app, as well as the `auth` backend plugin.

#### sign-in in frontend
- Sign-in is configured by providing a custom `SignInPage` app component. It will be rendered before any other routes in the app and is responsible for providing the identity of the current user. The `SignInPage` can render any number of pages and components, or just blank space with logic running in the background. In the end however it must provide a valid Backstage user identity through the `onSignInSuccess` callback prop, at which point the rest of the app is rendered.
- If you want to, you can use the `SignInPage` component that is provided by `@backstage/core-components`, which takes either a `provider` or `providers` (array) prop of `SignInProviderConfig` definitions.

> NOTE: You can configure sign-in to use a redirect flow with no pop-up by adding enableExperimentalRedirectFlow: true to the root of your app-config.yaml

#### sign-in in backend
- By default, every Backstage auth provider is configured only for the use-case of access delegation. This enables Backstage to request resources and actions from external systems on behalf of the user, for example re-triggering a build in CI.
- If you want to use an auth provider to sign in users, you need to explicitly configure it have sign-in enabled and also tell it how the external identities should be mapped to user identities within Backstage.

##### Backstage User Identity
- A user identity within Backstage is built up from two pieces of information:
1. a user entity reference
2. a set of ownership entity references
- When a user signs in, a Backstage token is generated with these two pieces of information, which is then used to identify the user within the Backstage ecosystem.

- The user entity reference should uniquely identify the logged in user in Backstage. It is encouraged that a matching user entity also exists within the Software Catalog, but it is not required. If the user entity exists in the catalog it can be used to store additional data about the user. There may even be some plugins that require this for them to be able to function.

- The ownership references are also entity references, and it is likewise encouraged that these entities exist within the catalog, but it is not a requirement. The ownership references are used to determine what the user owns, as a set of references that the user claims ownership though. For example, a user Jane (`user:default/jane`) might have the ownership references `user:default/jane`, `group:default/team-a`, and `group:default/admins`. Given these ownership claims, any entity that is marked as owned by either of `user:jane`, `team-a`, or `admins` would be considered owned by Jane.

- The ownership claims often contain the user entity reference itself, but it is not required. It is also worth noting that the ownership claims can also be used to resolve other relations similar to ownership, such as a claim for a `maintainer` or `operator` status.

- The Backstage token that encapsulates the user identity is a JWT. The user entity reference is stored in the sub claim of the payload, while the ownership references are stored in a custom ent claim. Both the user and ownership references should always be full entity references, as opposed to shorthands like just jane or user:jane.

##### Sign-in Resolvers
- Signing in a user into Backstage requires a mapping of the user identity from the third-party auth provider to a Backstage user identity. This mapping can vary quite a lot between different organizations and auth providers, and because of that there's no default way to resolve user identities. The auth provider that one wants to use for sign-in must instead be configured with a sign-in resolver, which is a function that is responsible for creating this user identity mapping.
- The input to the sign-in resolver function is the result of a successful log in with the given auth provider, as well as a context object that contains various helpers for looking up users and issuing tokens. There are also a number of built-in sign-in resolvers that can be used, which are covered a bit further down.
- Note that while it possible to configure multiple auth providers to be used for sign-in, you should take care when doing so. It is best to make sure that the different auth providers either do not have any user overlap, or that any users that are able to log in with multiple providers always end up with the same Backstage identity.
- 

## For Developing Plugins
### Identity for Plugin Developers
- For plugin developers, there is one main touchpoint for accessing the user identity: the `IdentityApi` exported by `@backstage/core-plugin-api` via the `identityApiRef`.
- The `IdentityApi` gives access to the signed-in user's identity in the frontend. It provides access to the user's entity reference, lightweight profile information, and a Backstage token that identifies the user when making authenticated calls within Backstage.
- When making calls to backend plugins, we recommend that the `FetchApi` is used, which is exported via the `fetchApiRef` from `@backstage/core-plugin-api`. The `FetchApi` will automatically include a Backstage token in the request, meaning there is no need to interact directly with the `IdentityApi`.

### Accessing Third Party Resources
- A common pattern for talking to third party services in Backstage is user-to-server requests, where short-lived OAuth Access Tokens are requested by plugins to authenticate calls to external services. These calls can be made either directly to the services or through a backend plugin or service.

- By relying on user-to-server calls we keep the coupling between the frontend and backend low, and provide a much lower barrier for plugins to make use of third party services. This is in comparison to for example a session-based system, where access tokens are stored server-side. Such a solution would require a much deeper coupling between the auth backend plugin, its session storage, and other backend plugins or separate services. A goal of Backstage is to make it as easy as possible to create new plugins, and an auth solution based on user-to-server OAuth helps in that regard.

- The method with which frontend plugins request access to third party services is through Utility APIs for each service provider. These are all suffixed with `*AuthApiRef`, for example `githubAuthApiRef`. For a full list of providers, see the @backstage/core-plugin-api reference.

---

## To Be Organized Later

## cache
- The Backstage backend and its built-in plugins are also able to leverage cache stores as a means of improving performance or reliability. Similar to how databases are supported, plugins receive logically separated cache connections, which are powered by Keyv under the hood.

- At this time of writing, Backstage can be configured to use one of three cache stores: memory, which is mainly used for local testing, memcache or Redis, which are cache stores better suited for production deployment. The right cache store for your Backstage instance will depend on your own run-time constraints and those required of the plugins you're running.

### Use memory for cache
```
backend:
  cache:
    store: memory
```

### Use memcache for cache
```
backend:
  cache:
    store: memcache
    connection: user:pass@cache.example.com:11211
```

### Use Redis for cache
```
backend:
  cache:
    store: redis
    connection: redis://user:pass@cache.example.com:6379
    useRedisSets: true
```
The useRedisSets flag is explained here.

## Containerization
The backend container can be built by running the following command:
```
yarn run build
yarn run build-image
```
This will create a container called ```example-backend```.

## Plugins
Home Template Plugin: https://github.com/backstage/backstage/tree/master/plugins/home

## Release & Versioning Policy
- The Backstage project is structured around two different release lines, a primary "main" release line, and a "next" release line that serves as a preview and pre-release of the next main-line release. Each of these release lines have their own release cadence and versioning policy.

### Main Release Line
- Release cadence: Monthly, specifically on the Tuesday before the third Wednesday of each month. The first release took place in March 2022.
- The version format is <major>.<minor>.<patch>, for example 1.3.0.

### Next Release Line
- Release cadence: Weekly, specifically on Tuesdays.
- The next release line is a weekly release of the project. Consuming these releases gives you early access to upcoming functionality in Backstage. There are however fewer guarantees around breaking changes in these releases, where moving from one release to the next may introduce significant breaking changes.

### Skew Policy
In order for Backstage to function properly the following versioning rules must be followed. The rules are referring to the Package Architecture.

- The versions of all the packages in the **Frontend App Core** must be from the same release, and it is recommended to keep **Common Tooling** on that release too.
- The Backstage dependencies of any given plugin should be from the same release. This includes the packages from **Common Libraries**, **Frontend Plugin Core**, and **Frontend Libraries**, or alternatively the **Backend Libraries**.
- There must be no package that is from a newer release than the **Frontend App Core** packages in the app.
- Frontend plugins with a corresponding backend plugin should be from the same release. The update to the backend plugin MUST be deployed before or together with the update to the frontend plugin.

### Node.JS Releases
- At any given point in time we support exactly two adjacent even-numbered releases of Node.js, for example v12 and v14.
- Once a new Node.js release becomes Active LTS we switch to support that release and the previous one. The switch is not immediate but done as soon as possible. You can find the Node.js version supported by each release in the **engines** field in the root **package.json** of a new app.

### TypeScript Releases
- The TypeScript release cadence is roughly every three months. An important aspect of the TypeScript versioning is that it does not follow semver (<major>.<minor>.<patch>).
- Our policy is to support the last 3 TypeScript versions, for example 4.8, 4.9, and 5.0. Converted to time, this means that we typically support the TypeScript version from the last six to nine months, depending on where in the TypeScript release window we are.
- For anyone maintaining their own Backstage project, this means that you should strive to bump to the latest TypeScript version at least every 6 months.

### PostgreSQL Releases
- The PostgreSQL versioning policy is to release a new major version every year with new features which is then supported for 5 years after its initial release.
- Our policy mirrors the PostgreSQL versioning policy - we will support the last 5 major versions. We will also test the newest and oldest versions in that range. For example, if the range we support is currently 12 to 16, then we would only test 12 and 16 explicitly.

---

# acknowledgment
## Contributors

APA 🖖🏻

## Links
- [Backstage repo](https://github.com/backstage/backstage)
- [Backstage Docs](https://backstage.io/docs/overview/what-is-backstage)
---
- How to easily create standardised software templates with Backstage: https://b-nova.com/en/home/content/easily-create-standardised-software-templates-with-backstage/
- Backstage by Example (Part 2): https://john-tucker.medium.com/backstage-by-example-part-2-b41cc12a5ad5
- Backstage Plugins by Example (Part 1): https://john-tucker.medium.com/backstage-plugins-by-example-part-1-a4737e21d256
---
- https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS142x+3T2022/home
- https://training.linuxfoundation.org/training/introduction-to-backstage-developer-portals-made-easy-lfs142x/
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
