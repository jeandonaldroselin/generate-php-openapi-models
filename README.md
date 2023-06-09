## OpenAPI Generator / Generate OpenApi Compatible Models

This repository was created to respond to the following problem => **The models generated by OpenAPI are not usable for documentation within a gateway API in PHP**.

The main reason is that theses models have been generated for the OpenAPI request but not for documentation purpose. [A very nice article](https://medium.com/xmglobal/including-external-openapi-models-in-your-own-openapi-definition-6c4c6507fe84) has been written here about this issue. However the given solution does not satisfy 100% my use case (I use JMS for serialization and the given solution was only working for Symfony Serializer).

After following theses instructions, you will be able to to generate **additional models** that are supported by NelmioApiDocBundle to document your PHP API gateways that will consume your microservices (or external microservices) using your generated PHP Client.

## 1 - Install the open api generator

I chose npm version of the generator, but you are free to choose the platform that suits you best.

```bash
npm i -g @openapitools/openapi-generator-cli
npm i -g npx
````

## 2 - Generate the PHP Client
1 - First, you will need to clone this project and place ourselves at the root of the project.

```bash
git clone https://github.com/jeandonaldroselin/generate-php-openapi-models.git
cd generate-php-openapi-models

```
2 -  Then run the PHP client generation command in this way. In real life, replace doc.json with your micro service's open api documentation.

```bash
npx @openapitools/openapi-generator-cli generate -i doc.json -g php -o php-client -c options-php.json -t php-templates
```

3 - Your php client has been generated in the php-client folder. You will find the model that will be used as an example in the directory **php-client/lib/Model/TokenOA.php**. In the following two examples, we assume that you have succeeded. to install the generated client in your PHP project and that it is located in the vendor folder with the namespace "OpenAPI\Client".

## 3 - Usage in a Symfony controller

```php
namespace App\Controller;

use Nelmio\ApiDocBundle\Annotation\Model;
use OpenAPI\Client\Model\TodoOA;
use OpenAPI\Client\Api\TodoApi;
use OpenAPI\Client\Configuration;
use OpenApi\Annotations as OA;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Annotation\Route;
// Use Symfony's serializer
use Symfony\Component\Serializer\SerializerInterface;
// or use JMS's serializer
// use JMS\Serializer\SerializerInterface;

class TodoController extends AbstractController
{
    /**
     * Get one todo using todo api client
     * 
     * @OA\Response(
     *     response=200,
     *     description="Get one todo item",
     *     @Model(type=TodoOA::class)
     * )
     * 
     * @OA\Tag(name="Todo")
     *
     * @param string $id
     * @param SerializerInterface $serializer
     * @return JsonResponse
     */
    #[Route('/api/todos/{id}', name: 'getOneTodo', methods: 'GET')]
    public function getOne(string $id,
                           SerializerInterface $serializer): JsonResponse
    {
        // forward the request to the todo api client
        $apiInstance = new TodoApi(
            null,
            (new Configuration())->setHost(
                // Replace "app.todo_api_base_url" by your parameter representing your api to call
                $this->getParameter('app.todo_api_base_url')
            )
        );
        return new JsonResponse(
            $apiInstance->getgetOneTodo($id)->__toString(),
            JsonResponse::HTTP_OK, ['accept' => 'json'], true
        );
    }

}
```

# 4 - Customization of models name in the NelmioApiDocBundleof a Symfony Project

EN: The name of the model that will appear in the nelmio documentation is for the moment **TodoOA** instead of **Todo** (which is the original name within the microservice). If this is blocking behavior for you, simply declare an alias in the nelmio_api_doc.yml file as follows:

```yaml
# config/packages/nelmio_api_doc.yaml
nelmio_api_doc:
    models:
        names:
            - { alias: Todo, type: Api\Client\Model\TodoOA }
```


# 5 - Ressources

As things never come out of nowhere, I was inspired by his resources to develop this template to generate php models that can be used by PHP API gateways : 

-> https://medium.com/xmglobal/including-external-openapi-models-in-your-own-openapi-definition-6c4c6507fe84

-> https://openapi-generator.tech/docs/templating/#models

-> https://zircote.github.io/swagger-php/reference/annotations.html#schema

-> https://symfony.com/bundles/NelmioApiDocBundle/current/alternative_names.html

-> https://github.com/OpenAPITools/openapi-generator/blob/master/modules/openapi-generator/src/main/resources/php/model_generic.mustache


---------

Enjoy ! If you meet any problem with this doc, don't hesitate to contact me at contact@siapep.fr.
