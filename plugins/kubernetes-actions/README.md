# Kubernetes actions for Backstage

This module provides [Backstage](https://backstage.io/) template [actions](https://backstage.io/docs/features/software-templates/builtin-actions) for [Kubernetes](https://kubernetes.io/docs/home/).

The following actions are currently supported in this module:

- Create a kubernetes namespace

## Getting started

1. Install the action package in your Backstage project

   ```bash
   yarn workspace backend add @janus-idp/backstage-scaffolder-backend-module-kubernetes
   ```

2. [Register](https://backstage.io/docs/features/software-templates/writing-custom-actions#registering-custom-actions) the Kubenretes actions by modifying the `packages/backend/src/plugins/scaffolder.ts` file from your project with the following changes:

   ```ts
   import { CatalogClient } from '@backstage/catalog-client';
   import { ScmIntegrations } from '@backstage/integration';
   import {
     createBuiltinActions,
     createRouter,
   } from '@backstage/plugin-scaffolder-backend';

   import { Router } from 'express';

   import { createKubernetesNamespaceAction } from '@janus-idp/backstage-scaffolder-backend-module-kubernetes';

   import type { PluginEnvironment } from '../types';

   export default async function createPlugin(
     env: PluginEnvironment,
   ): Promise<Router> {
     const catalogClient = new CatalogClient({
       discoveryApi: env.discovery,
     });

     const integrations = ScmIntegrations.fromConfig(env.config);

     const builtInActions = createBuiltinActions({
       integrations,
       catalogClient,
       config: env.config,
       reader: env.reader,
     });

     const actions = [
       ...builtInActions,
       createKubernetesNamespaceAction(catalogClient),
     ];

     return await createRouter({
       actions,
       logger: env.logger,
       config: env.config,
       database: env.database,
       reader: env.reader,
       catalogClient,
       identity: env.identity,
     });
   }
   ```

3. **Optional**: If you are doing the previous step for the first time, you also have to install the `@backstage/integration` package

   ```bash
   yarn workspace backend add @backstage/integration
   ```

4. Add the Kubernetes actions to your templates, see the [example](./examples/templates/01-kubernetes-template.yaml) file in this repository for complete usage examples

   ```yaml
   action: kubernetes:create-namespace
   id: create-kubernetes-namespace
   name: Create kubernetes namespace
   input:
     namespace: foo
     clusterRef: bar
     token: TOKEN
     skipTLSVerify: false
     caData: Zm9v
   ```

## Usage

### Action: kubernetes:create-namespace

#### Input

| Parameter Name |  Type   | Required | Description                                         | Example                           |
| -------------- | :-----: | :------: | --------------------------------------------------- | --------------------------------- |
| namespace      | string  |   Yes    | Kubernetes namespace name                           | foo                               |
| clusterRef     | string  |    No    | Cluster resource entity reference from the catalog  | bar                               |
| url            | string  |    No    | API url of the kubernetes cluster                   | <https://api.foo.redhat.com:6443> |
| token          | string  |    No    | Kubernetes API bearer token used for authentication |                                   |
| skipTLSVerify  | boolean |    No    | If true, ceritificate verification is skipped       | false                             |
| caData         | string  |    No    | Base64 encoded certificate data                     |                                   |

#### Output

This action doesn't have any outputs.

## Development

1. Add the local package dependency to the Backstage instance

   ```shell
   yarn workspace backend add file:./plugins/kubernetes-actions
   ```

2. [Register](#getting-started) the Kubernetes actions in your Backstage project
3. **Optional**: You can use the sample template from this repository and add it as `locations` in your `app-config.yaml` file

   ```yaml
   ---
   catalog:
     locations:
       - type: file
         target: ../../plugins/kubernetes-actions/examples/templates/01-kubernetes-template.yaml
         rules:
           - allow: [Template]
   ```

4. Run `yarn dev`
5. Make sure you have an available kubernetes cluster
6. Start using the Kubernetes actions in your templates
