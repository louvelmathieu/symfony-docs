.. index::
    single: Messenger; Getting results / Working with command & query buses

Getting Results from your Handler
---------------------------------

When a message is handled, the :class:`Symfony\\Component\\Messenger\\Middleware\\HandleMessageMiddleware`
adds a :class:`Symfony\\Component\\Messenger\\Stamp\\HandledStamp` for each object that handled the message.
You can use this to get the value returned by the handler(s):

.. configuration-block::

    .. code-block:: php

        use Symfony\Component\Messenger\MessageBusInterface;
        use Symfony\Component\Messenger\Stamp\HandledStamp;

        $envelope = $messageBus->dispatch(SomeMessage());

        // get the value that was returned by the last message handler
        $handledStamp = $envelope->last(HandledStamp::class);
        $handledStamp->getResult();

        // or get info about all of handlers
        $handledStamps = $envelope->all(HandledStamp::class);

A :class:`Symfony\\Component\\Messenger\\HandleTrait` also exists in order to ease
leveraging a Messenger bus for synchronous needs.
The :method:`Symfony\\Component\\Messenger\\HandleTrait::handle` method ensures
there is exactly one handler registered and returns its result.

Working with Command & Query Buses
----------------------------------

The Messenger component can be used in CQRS architectures where command & query
buses are central pieces of the application. Read Martin Fowler's
`article about CQRS`_ to learn more and
:doc:`how to configure multiple buses </messenger/multiple_buses>`.

As queries are usually synchronous and expected to be handled once,
getting the result from the handler is a common need.

To make this easy, you can leverage the ``HandleTrait`` in any class that has
a ``$messageBus`` property:

.. configuration-block::

    .. code-block:: php

        // src/Action/ListItems.php
        namespace App\Action;

        use App\Message\ListItemsQuery;
        use App\MessageHandler\ListItemsQueryResult;
        use Symfony\Component\Messenger\HandleTrait;
        use Symfony\Component\Messenger\MessageBusInterface;

        class ListItems
        {
            use HandleTrait;

            public function __construct(MessageBusInterface $messageBus)
            {
                $this->messageBus = $messageBus;
            }

            public function __invoke()
            {
                $result = $this->query(new ListItemsQuery(/* ... */));

                // Do something with the result
                // ...
            }

            // Creating such a method is optional, but allows type-hinting the result
            private function query(ListItemsQuery $query): ListItemsResult
            {
                return $this->handle($query);
            }
        }

Hence, you can use the trait to create command & query bus classes.
For example, you could create a special ``QueryBus`` class and inject it
wherever you need a query bus behavior instead of the ``MessageBusInterface``:

.. configuration-block::

    .. code-block:: php

        // src/MessageBus/QueryBus.php
        namespace App\MessageBus;

        use Symfony\Component\Messenger\Envelope;
        use Symfony\Component\Messenger\HandleTrait;
        use Symfony\Component\Messenger\MessageBusInterface;

        class QueryBus
        {
            use HandleTrait;

            public function __construct(MessageBusInterface $messageBus)
            {
                $this->messageBus = $messageBus;
            }

            /**
             * @param object|Envelope $query
             *
             * @return mixed The handler returned value
             */
            public function query($query)
            {
                return $this->handle($query);
            }
        }

.. _`article about CQRS`: https://martinfowler.com/bliki/CQRS.html
