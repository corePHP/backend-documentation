# Symfony Examples

This section shows **Symfony-specific examples** demonstrating how the architectural concepts
(Entity, Use Case, Service, Repository) are applied using real Symfony resources.

These examples intentionally show:
- Input deserialization
- Validation with Symfony Asserts
- Defensive use cases
- Use cases returning entities
- Controllers returning normalized output
- Realistic external service integration

---

## Controllers as Entry Points

In Symfony, controllers are **entry points**.

They are responsible for:
- Receiving HTTP requests
- Deserializing input
- Validating input
- Calling a use case
- Returning normalized output

Controllers do **not** contain business rules or orchestration logic.

---

### Example: Controller with Deserialization, Validation and Normalization

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;
    use Symfony\Component\Routing\Attribute\Route;
    use Symfony\Component\Validator\Validator\ValidatorInterface;
    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\Serializer\SerializerInterface;

    #[Route('/orders')]
    final class ShipOrderController extends AbstractController
    {
        public function __construct(
            private ShipOrder $useCase,
            private SerializerInterface $serializer,
            private ValidatorInterface $validator
        ) {}

        #[Route('/ship', methods: ['POST'])]
        public function __invoke(Request $request): Response
        {
            try {
                $input = $this->serializer->deserialize(
                    $request->getContent(),
                    ShipOrderInput::class,
                    'json'
                );

                $violations = $this->validator->validate($input);
                if (count($violations) > 0) {
                    return $this->json($violations, Response::HTTP_BAD_REQUEST);
                }

                $order = $this->useCase->execute($input);

                return $this->json(
                    $order,
                    Response::HTTP_OK,
                    [],
                    ['groups' => ['order:read']]
                );

            } catch (\DomainException $e) {
                return $this->json(
                    ['error' => $e->getMessage()],
                    Response::HTTP_UNPROCESSABLE_ENTITY
                );

            } catch (\Throwable $e) {
                return $this->json(
                    ['error' => 'Unexpected error'],
                    Response::HTTP_INTERNAL_SERVER_ERROR
                );
            }
        }
    }

The controller:
- Deserializes input
- Validates input
- Calls the use case
- Returns the entity using normalizers

---

## Input Objects and Validation

Input objects define the **contract** of a use case.

They:
- Are simple data holders
- Are validated using Symfony Asserts
- Contain no logic

---

### Example: Input DTO with Asserts

    use Symfony\Component\Validator\Constraints as Assert;

    final class ShipOrderInput
    {
        #[Assert\NotBlank]
        #[Assert\Type('integer')]
        public int $orderId;
    }

---

## Use Cases Returning Entities

Use cases **must return the entity** they modify or act upon.

This allows:
- Clear outcomes
- Consistent controller responses
- Flexible output formatting via normalizers

---

### Example: Defensive Use Case Returning an Entity

    final class ShipOrder
    {
        public function __construct(
            private OrderRepository $orders
        ) {}

        public function execute(ShipOrderInput $input): Order
        {
            $order = $this->orders->find($input->orderId);

            if ($order === null) {
                throw new DomainException('Order not found.');
            }

            $order->ship();

            $this->orders->save($order);

            return $order;
        }
    }

The use case:
- Validates existence
- Delegates rules to the entity
- Returns the modified entity

---

## Entities and Domain Rules

Entities enforce **business rules**, not input validation.

---

### Example: Doctrine Entity with Serialization Groups

    use Doctrine\ORM\Mapping as ORM;
    use Symfony\Component\Serializer\Annotation\Groups;

    #[ORM\Entity]
    final class Order
    {
        #[ORM\Id]
        #[ORM\GeneratedValue]
        #[ORM\Column]
        #[Groups(['order:read'])]
        private int $id;

        #[ORM\Column(length: 20)]
        #[Groups(['order:read'])]
        private string $status;

        public function ship(): void
        {
            if ($this->status !== 'paid') {
                throw new DomainException('Order cannot be shipped.');
            }

            $this->status = 'shipped';
        }
    }

---

## Repositories in Symfony

Repositories isolate Doctrine and expose **intent-focused methods**.

---

### Example: Doctrine Repository

    use Doctrine\ORM\EntityManagerInterface;

    final class DoctrineOrderRepository implements OrderRepository
    {
        public function __construct(
            private EntityManagerInterface $em
        ) {}

        public function find(int $id): ?Order
        {
            return $this->em->find(Order::class, $id);
        }

        public function save(Order $order): void
        {
            $this->em->persist($order);
            $this->em->flush();
        }
    }

---

## Services: Payment Providers with Authentication

Services represent **pluggable external capabilities**.

They:
- Are defined by interfaces
- Handle authentication internally
- Are interchangeable

---

### Example: Payment Service Interface

    interface PaymentProvider
    {
        public function charge(Money $amount, string $reference): void;
    }

---

### Example: Stripe Payment Provider

    final class StripePaymentProvider implements PaymentProvider
    {
        public function __construct(
            private string $apiKey
        ) {}

        public function charge(Money $amount, string $reference): void
        {
            // Authenticate using API key
            // Call Stripe API to charge
        }
    }

---

### Example: PayPal Payment Provider

    final class PaypalPaymentProvider implements PaymentProvider
    {
        public function __construct(
            private string $clientId,
            private string $clientSecret
        ) {}

        public function charge(Money $amount, string $reference): void
        {
            // Authenticate using client credentials
            // Call PayPal API to charge
        }
    }

---

## Using Payment Services in a Use Case

The use case decides **which provider to use** and **when to use it**.

---

### Example: Use Case Using a Payment Provider

    final class PayOrder
    {
        public function __construct(
            private OrderRepository $orders,
            private PaymentProvider $payments
        ) {}

        public function execute(PayOrderInput $input): Order
        {
            $order = $this->orders->find($input->orderId);

            if ($order === null) {
                throw new DomainException('Order not found.');
            }

            $this->payments->charge(
                $order->total(),
                (string) $order->id()
            );

            $order->markAsPaid();

            $this->orders->save($order);

            return $order;
        }
    }

---

## Summary

In Symfony:

- Controllers deserialize, validate, and return normalized entities
- Use cases orchestrate defensively and return entities
- Entities enforce business rules
- Repositories isolate Doctrine
- Services integrate external systems with authentication

Symfony provides infrastructure.  
The architecture defines responsibility.
