How to add a new API endpoint
=============================

Let’s say you want to add a new endpoint that will return just another list of earning points rules.
Here is a step-by-step guide how to achieve this.

First of all, you need to create a new controller in appropriate directory corresponding to the context (subdirectory of src/Ui/RestBundle/Controller in case of extending REST API).
Here is a sample code

.. code-block:: php

    <?php
    namespace OpenLoyalty\Ui\RestBundle\Controller\EarningRule;

    use FOS\RestBundle\Controller\Annotations\Route;
    use FOS\RestBundle\View\View;
    use Nelmio\ApiDocBundle\Annotation\Operation;
    use Swagger\Annotations as SWG;
    use Symfony\Component\HttpFoundation\Request;
    use FOS\RestBundle\Controller\FOSRestController;

    class GetList extends FOSRestController
    {
        /**
         * @Route(methods={"GET"}, name="app.earning_rule.index", path="/earningRule/list")
         *
         * @Operation(
         *     tags={"Earning Rule"},
         *     summary="Method will return all active earning rules.",
         *     @SWG\Response(
         *         response="200",
         *         description="Returned when successful"
         *     )
         * )
         *
         * @param Request $request
         *
         * @return View
         */
        public function __invoke(Request $request): View
        {
            return $this->view(['data' => ['some data']]);
        }
    }

.. note::

    According to ADR pattern it's important to implement ``__invoke`` method and to create only one controller for each endpoint.

@Route is an annotation to create a new route in Symfony Framework. The methods field specifies a list of methods that the endpoint will respond to.
The name is useful for creating links and redirection but not currently used as we’re implementing RESTful API.
The path is an endpoint URI. <https://symfony.com/doc/4.3/routing.html>`_.

@Operation is an annotation from NelmioDocApi bundle to create a documentation for our API. This documentation is
automatically generated from this annotation and available at ``http://openloyalty.localhost/doc``

More information about this bundle you can find `here <https://symfony.com/doc/current/bundles/NelmioApiDocBundle/index.html>`_.

@param and @return are standard comments for developers and self-explanatory.

Then we have an ``__invoke`` method implemented in our controller that takes HTTP request as an argument and return a json response.

Now, when we have a new controller, the last thing to do is register it in the framework. To do that, add a follow
line in ``src/Ui/Rest/Resources/config/routing.yml``

.. code-block:: yaml

    earning_rule:
        resource: '@OpenLoyaltyUiRest/Controller/EarningRule/'
        type: annotation

.. note:: It’s important to define this route before open_loyalty_core, not after, as Open Loyalty has an endpoint ``/api/earningRule/{earningRule}`` where ``{earningRule}`` is an variable and accepts any parameter, including ``list`` from our route.



***************
HTTP Responders
***************

It's a readable way of responding other types of data (other than json/rest api modeled data). In this approach a HTTP Response object should be returned by a Responder class which should contain only one public method ``__invoke()``
Here is an example of such use case:

.. code-block:: php

     <?php

     declare(strict_types=1);

     use Symfony\Component\HttpFoundation\Response;

     /**
      * Class InlineStreamResponder.
      */
     class InlineStreamResponder
     {
        /**
         * @param string $content
         * @param string $mimeType
         *
         * @return Response
         */
        public function __invoke(string $content, string $mimeType): Response
        {
            $response = new Response($content);
            $response->headers->set('Content-Disposition', 'inline');
            $response->headers->set('Content-Type', $mimeType);

            return $response;
        }
     }

After you have the responder's object injected (`inlineStreamResponder`) in your controller, it's simple to just return result of responders's `__invoke` method:

.. code-block:: php

            return $this->inlineStreamResponder->__invoke($content, $photo->getMime());

or call it directly:

.. code-block:: php

            return ($this->inlineStreamResponder)($content, $photo->getMime());

That’s it. Now you have a new API endpoint registered in Open Loyalty. You can go to the
``http://openloyalty.localhost/doc`` and try to call this endpoint.
By default, all our ``/api`` endpoints are behind a firewall. So if you want to use ``/api`` endpoints, you need to
be logged in as an administrator and use authorization token.

To see how Symfony firewall is configured check ``config/packages/security.yaml``
