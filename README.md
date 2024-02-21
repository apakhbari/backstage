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

## Structure
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


## Components
### 1-1 software catalog
#### Intro
- The Catalog’s objective is to map all software assets in your organization, including websites, APIs, libraries, and resources, in a centralized directory. This centralization is aimed at helping teams manage technology and enable discoverability. The Catalog tracks each asset's metadata, ownership, and dependencies, resulting in a software graph that surfaces orphaned entities.
- The Catalog is flexible enough to host a wide variety of software assets, known as entities in Backstage. Because a website is very different from a data processing pipeline, entities can be differentiated by kind. Moreover, even within kinds of entities, you can define types.
- The Catalog is powered by metadata stored in YAML files, which describe the kind, type, name, owner, and more details of a single entity per file. These files are commonly stored along their respective codebase, such that it gets updated frequently. Some entity kinds may have authoritative sources that are not represented in a codebase, for example, users and teams dictated by Okta.
- Tracking ownership and dependencies is one of the strongest use cases for the Catalog. These are also declared in the YAML file describing a software component. Only a single team can be the component owner, and this team must be registered in the Catalog as an entity as well. As for dependencies, Backstage lets you define how the component depends on another entity and what it exposes so others can consume it.

#### In Depth
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

![Entity lifecycle.png](./etc/Entity_lifecycle.png)

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

### 1-2 Scaffolder
#### Intro
- The Scaffolder provides your developers with the ability to execute software templates that initialize repositories with skeleton code and predefined settings.
- The Scaffolder is a perfect place for new engineers to jump into the development process right away. You could, for example, set up a template to Create Node/React Website in your Scaffolder, which sets up a repository with CI/CD and analytics baked in from the beginning. When the new developers use the software template, they’ll get a deploy-ready service that will allow them to familiarize themselves with the tools and feel productive with a few clicks instead of having to wander aimlessly through your tech ecosystem. 
- A software template is defined in a YAML file that specifies parameters and steps to execute. Backstage will generate a UI in the Scaffolder based on the parameters that you specify in your software template. For the steps, you can leverage built-in actions for common fetch operations, but you can also define your own. 
- Templates are stored in the Catalog under a template kind. Furthermore, all components initialized by the Scaffolder are automatically added to the Catalog. Therefore, there’s a virtuous cycle between the Scaffolder and the Catalog that promotes discoverability and standardization.


#### In Depth
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

### 1-3 TechDocs
#### Intro
- a centralized hub for all their documentation
- TechDocs is the framework’s documentation-as-code solution; it takes markdown files and transforms them into static pages
- TechDocs follows the same principle as the Catalog metadata files: stay close to the source code to stay accurate. TechDocs are written as markdown files in the repository where the entity that they document is kept. Then, the TechDocs Backstage plugin fetches these files from all services, generates static pages, and publishes them.

#### In Depth
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

### 1-4 Search
-  the most recent addition to Backstage’s framework.
-  earch allows developers to find information across their ecosystem by leveraging your search engine of preference and lets you customize how things are indexed and presented to the user.
-  Search is quite customizable. For starters, the plugin allows you to bring your own search engine, although ElasticSearch is the officially maintained engine. Search ships with a rudimentary query translator that turns the user’s input into a fully formed query, but you’re allowed to customize it to your engine and use cases better. You’re also welcome to customize the search results page and what each result looks like.
-  Under the hood, Search searches “documents” that represent entities, documentation pages, or any other thing that you put into Backstage. These documents are consumed through streams exposed by a Collator. Collators define what can be found by defining, indexing, and collecting documents. Currently, collators are available for the Catalog, TechDocs, and Stack Overflow. You can define your collators too.

### 1-5 k8s
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

---

## Setup Backstage Development Environment

### Set up the repo

1. Fork backstage into your GitHub user 
2. Clone the repo to your local workstation
```
git clone git@github.com:backstage/backstage.git
```
(This is an example from the Backstage repo)

3. cd to the backstage repo

4. Optional - checkout to latest stable version
```
git checkout v1.13.2
```

### Install & Configure NVM

NVM - Node Version Manager, nvm allows you to quickly install and use different versions of Node via the command line.

1. Install NVM
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```
2. Install Node 18
```
nvm install 18
```
3. Use Node version 18
```
nvm use 18
```
4. Set 18 as the default Node version
```
nvm alias default 18
```

### Install Yarn
1. Install Yarn
```
npm install --global yarn
```
2. Set Yarn to version 1
```
yarn set version 1.22.19
```
3. Verify the Yarn version
```
yarn --version
```

#### MAC x86 users: Verify you got xcrun
```bash
xcode-select --install
```

### Create your Backstage Application
1. Create a Backstage application
```
npx @backstage/create-app@latest
```
2. cd to your application directory (based on the given name)
```
cd guy-backstage/
```
3. Run backstage in a development mode
```
yarn dev
```

### Verify your application
1. Go to the Backstage UI (should open automatically after the `yarn dev` command)
2. Import a component using this file by going into [Register an existing component](http://localhost:3000/catalog-import) and import this file- `https://github.com/backstage/backstage/blob/master/catalog-info.yaml`
3. Review the example component


## Run Backstage On Kubernetes

### Build the container image
1. In your application directory, build your application
```
yarn install --frozen-lockfile
yarn tsc
yarn build:backend
```
2. Build the image
```
docker image build . -f packages/backend/Dockerfile --tag backstage:1.0.0
```

### Upload the image to your registry/kind cluster

This example is for Kind cluster, you can import the image to your docker repository

```
kind load docker-image backstage:1.0.0 --name local-single-node
```

### Deploy Postgres

We're going to use a local Postgres instance, you can use any postgres instance for it.

1. Create the backstage namespace
```
kubectl create ns backstage
```

2. Apply the manifests in the directory from this repo `portgres-resources`. It will create the Postgres resources required with a default username-password.
```
kubectl apply -f postgres-resources
```

from the file `pg-vol.yaml` before applying the manifests.

1. Verify the access to Postgres
```bash
export PG_POD=$(kubectl get pods -n backstage -o=jsonpath='{.items[0].metadata.name}')
```
```bash
kubectl exec -it --namespace=backstage $PG_POD -- /bin/bash

bash-5.1# psql -U $POSTGRES_USER
psql (13.2)
backstage=# \q
bash-5.1# exit
```

### Deploy Backstage on Kubernetes

1. Create the Backstage resources by preparing the files the apply them to the target cluster. You can find the instructions for it in the docs website as well - (link)[https://backstage.io/docs/deployment/k8s#creating-the-backstage-instance]


1. Edit `backstage-resources/bs-secret.yaml` with your github api token. token must have the permissions explained (here)[https://backstage.io/docs/integrations/github/locations/#token-scopes].

1. Apply the Backstage manifests
``` bash
kubectl apply -f backstage-resources
```

1. Check your running instance by port forwarding to it
``` bash
kubectl port-forward --namespace=backstage svc/backstage 8080:80
```

1. access the (backstage app)[http://127.0.0.1:8080/]


### Deploy the Backstage Kubernetes Plugin
1. Follow the instructions in the documentaion - (link)[https://backstage.io/docs/features/kubernetes/installation]


2. Configure the Kubernetes plugin in backstage by editing the `app-config.yaml`. Add this configuration to the end of the file.
```
kubernetes:
 serviceLocatorMethod:
   type: 'multiTenant'
 clusterLocatorMethods:
   - type: 'config'
     clusters:
       - url: kubernetes.default.svc.cluster.local:443
         name: local
         authProvider: 'serviceAccount'
         skipTLSVerify: false
         skipMetricsLookup: true
```

3. Build the app
```bash
yarn build:backend
```
4. Build the image
```
docker image build . -f packages/backend/Dockerfile --tag backstage:1.0.0
```
5. Push the image and run the backstage deployment with the new image

#### Configure your first app with Kubernetes

1. Run a sample app
```bash
kubectl apply -f example-application
```

2. Add the component configuration using by [Registering an existing component](http://localhost:3000/catalog-import) and load the `example-application/user-api-component.yaml`

3. Navigate to the new created app and go to the Kubernetes tab

## Cases
- FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
```
$ export NODE_OPTIONS=--max-old-space-size=4096
```

---

## Draft

## Cash
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


---

# acknowledgment
## Contributors

APA 🖖🏻

## Links
- [Backstage repo](https://github.com/backstage/backstage)
- [Backstage Docs](https://backstage.io/docs/overview/what-is-backstage)
---
- Platform Engineering Series : https://youtube.com/playlist?list=PLGVPcLSzJXQos1O18dvKoW2XSczz2I2lH&si=Jf4lDdaGW4GMLQYt
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
