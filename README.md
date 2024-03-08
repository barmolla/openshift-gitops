# Paso 1 (Instalación operador `Openshift GitOps)`

1. El operador crea un namespace `openshift-gitops` y dentro un CR `openshift-gitops` (hay que esperar..)

2. Hay que crear un `rbac` con los permisos para que `Openshift GitOps` pueda manipular los namespace.

```sh
oc create -f ocp01/core/rolebinding.yaml
```

# Paso 2 (Configuración)

Crear grupo cluster-admins

```sh
oc adm groups new cluster-admins
```

Agregar el usuario de la virtual al grupo creado

```
oc adm groups add-users cluster-admins opentlc-mgr
```

1. Configuramos `Openshift GitOps` registrando repositorio `openshift-gitops`

    ```sh
    oc create -f ocp01/core/apps-repository.yaml
    ```
    
2. [OPTIONAL] En caso de que sea necesario crear la llave SSH desde GitHub, seguir el siguiente tutorial: [Set up SSH key authentication](https://docs.github.com/es/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

3. Creamos un template de credenciales para todos los repositorios con la llave SSH generada en GitHub. Para ello se debe crear un nuevo repositorio y en vez de presionar [Conectar], se deben guardar las credenciales con [Save as Credentials Template].

    ![Configurar SSH Key](images/openshift-gitops-ssh-credentials.png)

    > Importante: Para que las credenciales tengan efecto en todos los repositorios que se agreguen, se debe crear el template con la ruta del repo hasta el nombre de la organización. Ej. **git@github.com:damianlezcano/**.

4. Registramos proyecto y aplicación en `Openshift GitOps`

    ```sh
    oc create -f ocp01/core/openshift-gitops-project.yaml
    oc create -f ocp01/core/applications.yaml
    ```
    _Esta aplicación es la encargada de monitorear las demás aplicaciónes_

# Paso 3 (Registramos aplicación de prueba)

1. Crear repo de manifiesto para la aplicación

    Para esto hay que crear un repositorio en github y alojar el contenido de la carpeta `example-gitops`

    ![](images/example-gitops.png)

2. Creamos el directorio del proyecto

    La información esta separada por `cluster` / `proyecto` / `aplicación`

    ```
    .
    ├── ocp01
    │  ├── proyectos
    │  │  ├── openshift-gitops
    │  │  │  └── rolebinding.yaml
    │  │  ├── project1
    │  │  │  ├── dev
    │  │  │  │  ├── apps
    │  │  │  │  │  └── example-app.yaml
    │  │  │  │  └── project
    │  │  │  │      ├── limit-range.yaml
    │  │  │  │      ├── namespace.yaml
    │  │  │  │      ├── resource-quota.yaml
    │  │  │  │      └── role-binding.yaml
    │  │  │  └── test
    │  │  │      ├── apps
    │  │  │      │  └── example-app.yaml
    │  │  │      └── project
    │  │  │          └── namespace.yaml
    │  │  └── project1-project.yaml
    │  └── repositories
    │      └── example-apps-repository.yaml
    └── ocp03
    ```

    Por ejemplo el proyecto `project1` del cluster `ocp01` esta compuesto de 2 ambientes (`dev` y `test`) y contiene la aplicación `example-app`

    En el ambiente `dev` del projecto `project1` se declara la creación del `namespace` y configuración de `limits`, `quota`, `rolebinding`.

    El cambio en el ambiente de `test` solo se creó el `namespace`

5. Verificación

    Si todo esta bien configurado, deberias ver en la consola de `Openshift GitOps` lo siguiente:

    ![](images/openshift-gitops-success1.png)

    Y en openshift que se crearon los `namespaces` con los objetos declarados en cada uno.

    ![](images/openshift-gitops-success2.png)

    _**Prueba en borrar alguno de estos 2 namespaces! `Openshift GitOps` los va regenerar nuevamente en forma automática. Observa la consola de `Openshift GitOps` para ver el progreso...**_
