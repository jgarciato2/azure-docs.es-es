---
title: Preparación de datos con grupos de Spark (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a asociar grupos de Spark para la preparación de datos con Azure Synapse y Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: nibaccam
author: nibaccam
ms.reviewer: nibaccam
ms.date: 03/02/2021
ms.custom: how-to, devx-track-python, data4ml
ms.openlocfilehash: 87e03b6aee122c5a26d4388ca8b570aa6cdf7b55
ms.sourcegitcommit: b4647f06c0953435af3cb24baaf6d15a5a761a9c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/02/2021
ms.locfileid: "101660525"
---
# <a name="attach-synapse-spark-pools-for-data-preparation-with-azure-synapse-preview"></a>Asociación de grupos de Spark de Synapse para la preparación de datos con Azure Synapse (versión preliminar)

En este artículo, aprenderá a asociar e iniciar un grupo de Apache Spark respaldado por [Azure Synapse](/synapse-analytics/overview-what-is.md) para la preparación de datos. 

>[!IMPORTANT]
> La integración de Azure Machine Learning y Azure Synapse se encuentra en versión preliminar. Las funciones que se presentan en este artículo emplean el paquete `azureml-synapse`, que contiene características en versión preliminar [experimentales](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py#stable-vs-experimental) que pueden cambiar en cualquier momento.

## <a name="azure-machine-learning-and-azure-synapse-integration-preview"></a>Integración de Azure Machine Learning y Azure Synapse (versión preliminar)

La integración de Azure Synapse con Azure Machine Learning (versión preliminar) le permite asociar un grupo de Apache Spark respaldado por Azure Synapse para la exploración y preparación interactivas de datos. Con esta integración, puede contar con un proceso dedicado para la preparación de datos a gran escala, todo dentro del mismo cuaderno de Python que usa para entrenar los modelos de Machine Learning.

## <a name="prerequisites"></a>Requisitos previos

* [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md?tabs=python).

* [Cree un área de trabajo de Synapse en Azure Portal](../synapse-analytics/quickstart-create-workspace.md).

* [Cree un grupo de Apache Spark mediante Azure Portal, herramientas web o Synapse Studio](../synapse-analytics/quickstart-create-apache-spark-pool-portal.md).

* [Instale el SDK de Python de Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py) que incluye el paquete `azureml-synapse` (versión preliminar). 
    * También puede instalarlo usted mismo, pero debe tener en cuenta que solo es compatible con las versiones 1.20 o posteriores del SDK. 
        ```python
        pip install azureml-synapse
        ```

## <a name="link-machine-learning-workspace-and-synapse-assets"></a>Vinculación del área de trabajo de Machine Learning y los recursos de Synapse

Para poder asociar un grupo de Spark de Synapse para la preparación de datos, las áreas de trabajo de Azure Machine Learning y de Azure Synapse deben estar vinculadas. 

Puede vincular ambas áreas de trabajo mediante el [SDK de Python](#link-sdk) o [Estudio de Azure Machine Learning](#link-studio). 

> [!IMPORTANT]
> Para vincular correctamente el área de trabajo de Synapse, debe tener el rol **Propietario** en ella. Compruebe el acceso en [Azure Portal](https://ms.portal.azure.com/).
>
> Si no tiene el rol **Propietario** en el área de trabajo de Synapse, pero quiere usar un servicio vinculado existente, consulte [Obtención de un servicio vinculado existente](#get-an-existing-linked-service).


<a name="link-sdk"></a>
### <a name="link-workspaces-with-the-python-sdk"></a>Vinculación de áreas de trabajo con el SDK de Python

En el código siguiente se usan las clases [`LinkedService`](/python/api/azureml-core/azureml.core.linked_service.linkedservice?preserve-view=true&view=azure-ml-py) y [`SynapseWorkspaceLinkedServiceConfiguration`](/python/api/azureml-core/azureml.core.linked_service.synapseworkspacelinkedserviceconfiguration?preserve-view=true&view=azure-ml-py) para: 

* Vincular el área de trabajo de Machine Learning `ws` con el área de trabajo de Azure Synapse. 
* Registrar el área de trabajo de Synapse con Azure Machine Learning como servicio vinculado.

``` python
import datetime  
from azureml.core import Workspace, LinkedService, SynapseWorkspaceLinkedServiceConfiguration

# Azure Machine Learning workspace
ws = Workspace.from_config()

#link configuration 
synapse_link_config = SynapseWorkspaceLinkedServiceConfiguration(
    subscription_id=ws.subscription_id,
    resource_group= 'your resource group',
    name='mySynapseWorkspaceName')

# Link workspaces and register Synapse workspace in Azure Machine Learning
linked_service = LinkedService.register(workspace = ws,              
                                            name = 'synapselink1',    
                                            linked_service_config = synapse_link_config)
```
> [!IMPORTANT] 
> Se crea una identidad administrada, `system_assigned_identity_principal_id`, para cada servicio vinculado. A esta identidad administrada se le debe conceder el rol **Administrador de Apache Spark de Synapse** del área de trabajo de Synapse antes de iniciar la sesión de este servicio. [Asigne el rol Administrador de Apache Spark de Synapse a la identidad administrada en Synapse Studio](../synapse-analytics/security/how-to-manage-synapse-rbac-role-assignments.md).
>
> Utilice `LinkedService.get('<your-mlworkspace-name>', '<linked-service-name>')` para localizar el valor `system_assigned_identity_principal_id` de un servicio vinculado específico.

<a name="link-studio"></a>
### <a name="link-workspaces-via-studio"></a>Vinculación de áreas de trabajo por medio de Estudio

Vincule el área de trabajo de Machine Learning y el área de trabajo de Synapse mediante Estudio de Azure Machine Learning con los pasos siguientes: 

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).
1. En la sección **Administrar** del panel izquierdo, seleccione **Servicios vinculados**.
1. Seleccione **Agregar integración**.
1. Rellene los campos del formulario **Vincular área de trabajo**.

   |Campo| Descripción    
   |---|---
   |Nombre| Indique un nombre para el servicio vinculado. Este nombre es el que se usará para hacer referencia a este servicio vinculado concreto.
   |Nombre de suscripción | Seleccione el nombre de la suscripción que está asociada con el área de trabajo de Machine Learning. 
   |Área de trabajo de Synapse | Seleccione el área de trabajo de Synapse con la que desea establecer el vínculo. 
   
1. Seleccione **Siguiente** para abrir el formulario **Selección de los grupos de Spark (opcional)** . En este formulario, seleccione el grupo de Spark de Synapse que se va a asociar al área de trabajo.

1. Seleccione **Siguiente** para abrir el formulario **Revisión** y comprobar sus selecciones. 
1. Seleccione **Crear** para completar el proceso de creación del servicio vinculado.

## <a name="get-an-existing-linked-service"></a>Obtención de un servicio vinculado existente

Para recuperar y usar un servicio vinculado existente, se necesitan permisos de **usuario o colaborador** en el área de trabajo de Synapse.

En este ejemplo se recupera un servicio vinculado existente, `synapselink1`, del área de trabajo `ws` con el método [`get()`](/python/api/azureml-core/azureml.core.linkedservice?preserve-view=true&view=azure-ml-py#get-workspace--name-).
```python
linked_service = LinkedService.get(ws, 'synapselink1')
```

### <a name="manage-linked-services"></a>Administración de servicios vinculados

Para desvincular las áreas de trabajo, use el método `unregister()`.

``` python
linked_service.unregister()
```

Consulte todos los servicios vinculados asociados con el área de trabajo de Machine Learning. 

```python
LinkedService.list(ws)
```
 
## <a name="attach-synapse-spark-pool-as-a-compute"></a>Asociación de un grupo de Spark de Synapse como proceso

Después de vincular las áreas de trabajo, asocie un grupo de Spark de Synapse como recurso de proceso dedicado para las tareas de preparación de datos. 

Puede asociar grupos de Spark de Synapse mediante estas opciones:
* Azure Machine Learning Studio
* [Plantillas de Azure Resource Manager (ARM)](https://github.com/Azure/azure-quickstart-templates/blob/master/101-machine-learning-linkedservice-create/azuredeploy.json)
* SDK de Python 

Siga estos pasos para asociar un grupo de Spark de Synapse mediante Studio. 

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).
1. En la sección **Administrar** del panel izquierdo, seleccione **Servicios vinculados**.
1. Seleccione el área de trabajo de Synapse.
1. Seleccione **Grupos de Spark adjuntos** en la parte superior izquierda. 
1. Seleccione **Adjuntar**. 
1. Seleccione el grupo de Spark de Synapse en la lista y proporcione un nombre.  
    1. Esta lista identifica los grupos de Spark de Synapse disponibles que se pueden asociar al proceso. 
    1. Para crear un nuevo grupo de Spark de Synapse, consulte [Creación de un grupo de Apache Spark con Synapse Studio](../synapse-analytics/quickstart-create-apache-spark-pool-portal.md).
1. Seleccione **Asociar selección**. 


También puede utilizar el **SDK de Python** para asociar un grupo de Spark de Synapse. 

El código siguiente: 
1. Configura SynapseCompute con:

   1. El objeto LinkedService, `linked_service`, que creó o recuperó en el paso anterior. 
   1. El tipo de destino de proceso que quiere asociar, `SynapseSpark`.
   1. El nombre del grupo de Spark de Synapse. Debe coincidir con un grupo de Apache Spark existente que se encuentre en el área de trabajo de Synapse.
   
1. Crea un objeto ComputeTarget de aprendizaje automático pasando: 
   1. El área de trabajo de Machine Learning que desea usar, `ws`.
   1. El nombre con el que quiere hacer referencia al proceso en el área de trabajo de Machine Learning. 
   1. El valor de attach_configuration que especificó al configurar SynapseCompute.
       1. La llamada a ComputeTarget.attach() es asincrónica, por lo que el ejemplo crea un bloqueo hasta que se completa la llamada.

```python
from azureml.core.compute import SynapseCompute, ComputeTarget

attach_config = SynapseCompute.attach_configuration(linked_service, #Linked synapse workspace alias
                                                    type='SynapseSpark', #Type of assets to attach
                                                    pool_name="<Synapse Spark pool name>") #Name of Synapse spark pool 

synapse_compute = ComputeTarget.attach(workspace= ws,                
                                       name='<Synapse Spark pool alias in Azure ML>', 
                                       attach_configuration=attach_config
                                      )

synapse_compute.wait_for_completion()
```

Compruebe que el grupo de Spark de Synapse está asociado.

```python
ws.compute_targets['Synapse Spark pool alias']
```

## <a name="launch-synapse-spark-pool-for-data-preparation-tasks"></a>Inicio del grupo de Spark de Synapse para tareas de preparación de datos

Puede especificar un [entorno de Azure Machine Learning](concept-environments.md) para usarlo durante la sesión de Synapse. Solo se aplicarán las dependencias de Conda especificadas en el entorno. No se admite la imagen de Docker.

>[!WARNING]
>  Los grupos de Spark de Synapse no admiten las dependencias de Python especificadas en las dependencias de Conda del entorno. Actualmente, solo se admiten las versiones fijas de Python. Compruebe la versión de Python incluyendo `sys.version_info` en el script.

En el código siguiente se crea el entorno, `myenv`, que instala `azureml-core` versión 1.20.0 y `numpy` versión 1.17.0 antes de que se inicie la sesión. Después puede incluir este entorno en la instrucción `start` de la sesión de Synapse.

```python

from azureml.core import Workspace, Environment

# creates environment with numpy and azureml-core dependencies
ws = Workspace.from_config()
env = Environment(name="myenv")
env.python.conda_dependencies.add_pip_package("azureml-core==1.20.0")
env.python.conda_dependencies.add_conda_package("numpy==1.17.0")
env.register(workspace=ws)
```

Para comenzar la preparación de datos con el grupo de Spark de Synapse, especifique el nombre de este grupo y proporcione el identificador de la suscripción, el grupo de recursos del área de trabajo de Machine Learning, el nombre del área de trabajo de Machine Learning y el entorno que se usará durante la sesión de Synapse. 

> [!IMPORTANT]
> Para seguir usando el grupo de Spark de Synapse, debe indicar qué recurso de proceso se va a usar en las tareas de preparación de datos, con `%synapse` para líneas de código únicas y `%%synapse` para varias líneas. 

```python
%synapse start -c SynapseSparkPoolAlias -s AzureMLworkspaceSubscriptionID -r AzureMLworkspaceResourceGroupName -w AzureMLworkspaceName -e myenv
```

Una vez iniciada la sesión, puede comprobar sus metadatos.

```python
%synapse meta
```

## <a name="load-data-from-storage"></a>Carga de datos desde el almacenamiento

Una vez iniciada la sesión de Spark de Synapse, lea los datos que desea preparar. La carga de datos es compatible con Azure Blob Storage y Azure Data Lake Storage Generation 1 y Generation 2.

Los datos de estos servicios de almacenamiento se pueden cargar de dos maneras: 

* Carga directa de los datos desde el almacenamiento mediante su ruta de acceso del sistema de archivos distribuido de Hadoop (HDFS).

* Lectura de los datos de un [conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md) existente.

Para acceder a estos servicios de almacenamiento, necesita permisos de **lector de datos de Storage Blob**. Si tiene previsto escribir datos de nuevo en estos servicios de almacenamiento, necesitará permisos de **colaborador de datos de Storage Blob**. [Obtenga más información sobre los permisos y roles de almacenamiento](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues).

### <a name="load-data-with-hadoop-distributed-files-system-hdfs-path"></a>Carga de datos con la ruta de acceso del sistema de archivos distribuido de Hadoop (HDFS)

Para cargar y leer datos de almacenamiento con la ruta de acceso de HDFS correspondiente, debe tener preparadas las credenciales de autenticación de acceso a datos. Estas credenciales varían según el tipo de almacenamiento.  

En el código siguiente se muestra cómo se leen datos de **Azure Blob Storage** en una trama de datos de Spark con la clave de acceso o el token de firma de acceso compartido (SAS). 

```python
%%synapse

# setup access key or SAS token
sc._jsc.hadoopConfiguration().set("fs.azure.account.key.<storage account name>.blob.core.windows.net", "<access key>")
sc._jsc.hadoopConfiguration().set("fs.azure.sas.<container name>.<storage account name>.blob.core.windows.net", "sas token")

# read from blob 
df = spark.read.option("header", "true").csv("wasbs://demo@dprepdata.blob.core.windows.net/Titanic.csv")
```

En el código siguiente se muestra cómo se leen datos de **Azure Data Lake Storage Generation 1 (ADLS Gen1)** con las credenciales de la entidad de servicio. 

```python

# setup service principal which has access of the data
sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.access.token.provider.type","ClientCredential")

sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.client.id", "<client id>")

sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.credential", "<client secret>")

sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.refresh.url",
https://login.microsoftonline.com/<tenant id>/oauth2/token)

df = spark.read.csv("adl://<storage account name>.azuredatalakestore.net/<path>")

```

En el código siguiente se muestra cómo se leen datos de **Azure Data Lake Storage Generation 2 (ADLS Gen2)** con las credenciales de la entidad de servicio. 

```python
# setup service principal which has access of the data
sc._jsc.hadoopConfiguration().set("fs.azure.account.auth.type.<storage account name>.dfs.core.windows.net","OAuth")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth.provider.type.<storage account name>.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth2.client.id.<storage account name>.dfs.core.windows.net", "<client id>")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth2.client.secret.<storage account name>.dfs.core.windows.net", "<client secret>")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth2.client.endpoint.<storage account name>.dfs.core.windows.net",
https://login.microsoftonline.com/<tenant id>/oauth2/token)


df = spark.read.csv("abfss://<container name>@<storage account>.dfs.core.windows.net/<path>")

```

### <a name="read-in-data-from-registered-datasets"></a>Lectura de datos de conjuntos de datos registrados

También puede obtener un conjunto de datos registrado existente en el área de trabajo y realizar la preparación de datos convirtiéndolo en una trama de datos de Spark.  

En el ejemplo siguiente se obtiene un objeto TabularDataset registrado, `blob_dset`, que hace referencia a los archivos del almacenamiento de blobs y lo convierte en una trama de datos de Spark. Al convertir los conjuntos de datos en una trama de datos de Spark, puede aprovechar las bibliotecas de preparación y exploración de datos `pyspark`.  

``` python

%%synapse
from azureml.core import Workspace, Dataset

dset = Dataset.get_by_name(ws, "blob_dset")
spark_df = dset.to_spark_dataframe()
```

## <a name="perform-data-preparation-tasks"></a>Realización de tareas de preparación de datos

Después de recuperar y explorar los datos, puede llevar a cabo las tareas de preparación de datos.

En el código siguiente, se amplía el ejemplo de HDFS de la sección anterior y se filtran los datos de la trama de datos de Spark, `df`, en función de la columna **Survivor** y se agrupa la lista según el valor de **Age**.

```python
%%synapse
from pyspark.sql.functions import col, desc

df.filter(col('Survived') == 1).groupBy('Age').count().orderBy(desc('count')).show(10)

df.show()

```

## <a name="save-data-to-storage-and-stop-spark-session"></a>Almacenamiento de datos y detención de la sesión de Spark

Una vez completada la exploración y la preparación de los datos, almacene los datos preparados para poder usarlos más adelante en su cuenta de almacenamiento de Azure.

En el ejemplo siguiente, los datos preparados se escriben en Azure Blob Storage y sobrescriben el archivo `Titanic.csv` original del directorio `training_data`. Para volver a escribir en el almacenamiento, necesita permisos de **colaborador de datos de Blob Storage**. [Obtenga más información sobre los permisos y roles de almacenamiento](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues).

```python
%% synapse

df.write.format("csv").mode("overwrite").save("wasbs://demo@dprepdata.blob.core.windows.net/training_data/Titanic.csv")
```

Cuando haya terminado de preparar los datos y haya guardado los datos preparados en el almacenamiento, detenga el uso del grupo de Spark de Synapse con el siguiente comando.

```python
%synapse stop
```

## <a name="create-dataset-to-represent-prepared-data"></a>Creación de conjuntos de datos para representar los datos preparados

Cuando esté listo para usar los datos preparados para el entrenamiento de modelos, conéctese al almacenamiento con un [almacén de datos de Azure Machine Learning](how-to-access-data.md) y especifique los archivos que desea usar con un [conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md).

Consulte el ejemplo de código siguiente:

* En él se supone que ya ha creado un almacén de datos que se conecta al servicio de almacenamiento en el que guardó los datos preparados.  
* Obtiene el almacén de datos existente, `mydatastore`, del área de trabajo, `ws`, con el método get().
* Crea un objeto [FileDataset](how-to-create-register-datasets.md#filedataset), `train_ds`, que hace referencia a los archivos de datos preparados ubicados en el directorio `training_data` de `mydatastore`.  
* Crea la variable `input1`, que se puede usar más adelante para que los archivos de datos del conjunto de datos `train_ds` estén disponibles para un destino de proceso.

```python
from azureml.core import Datastore, Dataset

datastore = Datastore.get(ws, datastore_name='mydatastore')

datastore_paths = [(datastore, '/training_data/')]
train_ds = Dataset.File.from_files(path=datastore_paths, validate=True)
input1 = train_ds.as_mount()

```

## <a name="example-notebook"></a>Cuaderno de ejemplo

Consulte en este [cuaderno completo](../synapse-analytics/overview-what-is.md) un ejemplo de código detallado sobre cómo realizar la preparación de datos y el entrenamiento del modelo desde un solo cuaderno con Azure Synapse y Azure Machine Learning.

## <a name="next-steps"></a>Pasos siguientes

* [Entrenamiento de un modelo](how-to-set-up-training-targets.md).
* [Entrenamiento de modelos con conjuntos de datos de Azure Machine Learning](how-to-train-with-datasets.md)
* [Creación de un conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md).

