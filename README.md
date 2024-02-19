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
### Core
Deployed by default (e.g. software catalog)
1. SOftware catalog: all info about system & organization
2. Tech Docs: Documentatin inside backstage
3. Search: search across all backstage portal
4. Service Template: bootstrap services in backstage

### App (Integrations)
Extend capabilities of backstage. Deployed but require configuration (e.g. SSO / Analytics / Git)

### Plugins
Require installation, integration & configuration to your backstage application manually

Types of plugins:
- Service backed Plugins: Service is in same cluster/entity. deployed on different service in same system
- proxy backed plugin: redirect to proxy, which fetch info from some service outside of this cluster/entity

## Config files
- app-config.yaml --> app config. all settings are here, plugins are being add here
- catalog --> holds components in your system

---
# Backstage tutorial Linux Foundation
https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS142x+3T2022/home

https://training.linuxfoundation.org/training/introduction-to-backstage-developer-portals-made-easy-lfs142x/

- Backstage’s core is composed of around 25 packages, which include a CLI, utility tools, API definitions, themes, and helpers. But what really makes up Backstage are the more than 150 open source plugin packages available, which include the framework’s main features.
- Backstage offers five core functionalities:
1. software catalog
2. software templates
3. a documentation generator
4. a Kubernetes cluster visualizer
5. cross-ecosystem search capabilities
## Components
### software catalog
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
- In the snippet above, notice that apiVersion, kind, and metadata.name are mandatory for all entities. And for a component, such as a website from the example, you must also specify type, lifecycle, and owner in spec.
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


### Scaffolder
- The Scaffolder provides your developers with the ability to execute software templates that initialize repositories with skeleton code and predefined settings.
- The Scaffolder is a perfect place for new engineers to jump into the development process right away. You could, for example, set up a template to Create Node/React Website in your Scaffolder, which sets up a repository with CI/CD and analytics baked in from the beginning. When the new developers use the software template, they’ll get a deploy-ready service that will allow them to familiarize themselves with the tools and feel productive with a few clicks instead of having to wander aimlessly through your tech ecosystem. 
- A software template is defined in a YAML file that specifies parameters and steps to execute. Backstage will generate a UI in the Scaffolder based on the parameters that you specify in your software template. For the steps, you can leverage built-in actions for common fetch operations, but you can also define your own. 
- Templates are stored in the Catalog under a template kind. Furthermore, all components initialized by the Scaffolder are automatically added to the Catalog. Therefore, there’s a virtuous cycle between the Scaffolder and the Catalog that promotes discoverability and standardization.

### TechDocs
- a centralized hub for all their documentation
- TechDocs is the framework’s documentation-as-code solution; it takes markdown files and transforms them into static pages
- TechDocs follows the same principle as the Catalog metadata files: stay close to the source code to stay accurate. TechDocs are written as markdown files in the repository where the entity that they document is kept. Then, the TechDocs Backstage plugin fetches these files from all services, generates static pages, and publishes them.

### k8s
- he Kubernetes plugin is tied to the Catalog. It shows information about the clusters associated with a service registered in the catalog. To enable it, you must tell Backstage how to discover your clusters, whether that is by reading information that exists already in the Catalog or by fetching it directly from GKE or another custom Kubernetes cluster supplier.

### Search
-  the most recent addition to Backstage’s framework.
-  earch allows developers to find information across their ecosystem by leveraging your search engine of preference and lets you customize how things are indexed and presented to the user.
-  Search is quite customizable. For starters, the plugin allows you to bring your own search engine, although ElasticSearch is the officially maintained engine. Search ships with a rudimentary query translator that turns the user’s input into a fully formed query, but you’re allowed to customize it to your engine and use cases better. You’re also welcome to customize the search results page and what each result looks like.
-  Under the hood, Search searches “documents” that represent entities, documentation pages, or any other thing that you put into Backstage. These documents are consumed through streams exposed by a Collator. Collators define what can be found by defining, indexing, and collecting documents. Currently, collators are available for the Catalog, TechDocs, and Stack Overflow. You can define your collators too.

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



---

[Backstage Docs](https://backstage.io/docs/overview/what-is-backstage)

<br>
<br>

## Setup Backstage Development Environment

<br>
<br>

### Set up the repo

1. Fork backstage into your GitHub user [Backstage repo](https://github.com/backstage/backstage)
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

<br>
<br>

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

<br>
<br>

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

<br>
<br>

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

<br>
<br>

### Verify your application
1. Go to the Backstage UI (should open automatically after the `yarn dev` command)
2. Import a component using this file by going into [Register an existing component](http://localhost:3000/catalog-import) and import this file- `https://github.com/backstage/backstage/blob/master/catalog-info.yaml`
3. Review the example component

<br>
<br>

## Run Backstage On Kubernetes

<br>
<br>

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

<br>
<br>

### Upload the image to your registry/kind cluster

This example is for Kind cluster, you can import the image to your docker repository

```
kind load docker-image backstage:1.0.0 --name local-single-node
```

<br>
<br>

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

<br>
<br>

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

<br>
<br>

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


### Deploy on GKE
1. Remove the sections that create local volumes
```bash
  labels:
    type: local
...
    storageClassName: manual
```
2. Change the pg deployment by deleting the next line:
```bash
subPath: postgres
```

3. Upload your image to some container registry

4. Change the image in the bs-deploy.yaml to your image and apply it

5. Create an external service
```bash
kubectl apply -f gke-resources
```
## Cases
- FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
```
$ export NODE_OPTIONS=--max-old-space-size=4096
```

# acknowledgment
## Contributors

APA 🖖🏻

## Links
Platform Engineering Series : https://youtube.com/playlist?list=PLGVPcLSzJXQos1O18dvKoW2XSczz2I2lH&si=Jf4lDdaGW4GMLQYt

How to easily create standardised software templates with Backstage: https://b-nova.com/en/home/content/easily-create-standardised-software-templates-with-backstage/

Backstage by Example (Part 2): https://john-tucker.medium.com/backstage-by-example-part-2-b41cc12a5ad5

Backstage Plugins by Example (Part 1): https://john-tucker.medium.com/backstage-plugins-by-example-part-1-a4737e21d256


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
