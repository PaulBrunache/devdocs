---
layout: default
group: payments-integrations
subgroup: A_gateway
title: Response Handler
menu_title: Response Handler
menu_node: 
menu_order: 7
version: 2.0
github_link: payments-integrations/payment-gateway/response-handler.md
---

Response Handler is the component of Magento payment provider gateway, that processes payment provider response. Typically, the response requires one of the following actions:

- Modify the order status
- Save information that was provided in a transaction response
- Send an email

The response handler only modifies the order state, based on the payment gateway response. It does not perform any other required actions. 

## Interface

Basic interface for a response handler is [`Magento\Payment\Gateway\Response\HandlerInterface`]({{site.mage2000url}}app/code/Magento/Payment/Gateway/Response/HandlerInterface.php)


### Useful implementations

`\Magento\Payment\Gateway\Response\HandlerChain` might be used as a basic container of response handlers, handling different parts.

### Example

Example of a simple response handler ([`app/code/Magento/Braintree/Gateway/Response/PayPalDetailsHandler.php`]({{site.mage2100url}}app/code/Magento/Braintree/Gateway/Response/PayPalDetailsHandler.php)):

{% highlight php startinline=1 %}
class PayPalDetailsHandler implements HandlerInterface
{
    const PAYMENT_ID = 'paymentId';

    const PAYER_EMAIL = 'payerEmail';

    /**
     * @var SubjectReader
     */
    private $subjectReader;

    /**
     * Constructor
     *
     * @param SubjectReader $subjectReader
     */
    public function __construct(SubjectReader $subjectReader)
    {
        $this->subjectReader = $subjectReader;
    }

    /**
     * @inheritdoc
     */
    public function handle(array $handlingSubject, array $response)
    {
        $paymentDO = $this->subjectReader->readPayment($handlingSubject);

        /** @var \Braintree\Transaction $transaction */
        $transaction = $this->subjectReader->readTransaction($response);

        /** @var OrderPaymentInterface $payment */
        $payment = $paymentDO->getPayment();

        $payPal = $this->subjectReader->readPayPal($transaction);
        $payment->setAdditionalInformation(self::PAYMENT_ID, $payPal[self::PAYMENT_ID]);
        $payment->setAdditionalInformation(self::PAYER_EMAIL, $payPal[self::PAYER_EMAIL]);
    }
}
{% endhighlight %}

(the code sample is from Magento CE v2.1. Although the payment provider gateway was added in v2.0, the particular default implementation using the gateway were added in v2.1)