## Capture funds with be2bill payment

### Step 1. Download be2bill payum lib

Add the following lines in your `composer.json` file:

```json
{
    "require": {
        "payum/be2bill": "dev-master"
    }
}
```

_**Note:** You may want to adapt this line to use a specific version._

Now, run composer.phar to download the bundle:

```bash
$ php composer.phar install
```

_**Note:** You can immediately start using it. The autoloading files have been generated by composer and already included to the app autoload file._

### Step 2: Basic configuration

#### 1. Configure payment context
```yaml
#app/config/config.yml

payum:
    contexts:
        your_context_name:
            be2bill_payment:
                api:
                    options:
                        identifier: 'get this from gateway'
                        password: 'get this from gateway'
                        sandbox: true
```

**Warning:**

> You have to changed this name `your_context_name` to something related to your domain, for example `post_a_job_with_be2bill`

#### 2-a. Configure doctrine storage

Extend payment instruction class with added id property:

```php
<?php
//src/Acme/DemoBundle/Entity

namespace AcmeDemoBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use Payum\Be2Bill\Bridge\Doctrine\Entity\PaymentInstruction;

/**
 * @ORM\Entity
 */
class Be2BillPaymentInstruction extends PaymentInstruction
{
    /**
     * @ORM\Column(name="id", type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="IDENTITY")
     */
    protected $id;
}
```

and configure storage to use this model:

```yml
#app/config/config.yml

payum:
    contexts:
        your_context_name:
            doctrine_storage:
                driver: orm
                model_class: AcmeDemoBundle\Entity\Be2BillPaymentInstruction

doctrine:
    orm:
        entity_managers:
            default:
                mappings: 
                    payum_be2bill:
                        is_bundle: false
                        type: xml 
                        dir: %kernel.root_dir%/../vendor/payum/be2bill/src/Payum/Be2Bill/Bridge/Doctrine/Resources/mapping
                        prefix: Payum\Be2Bill\Bridge\Doctrine\Entity
```

#### 2-b. Configure filesystem storage

Extend payment instruction class with added `id` property:

```php
<?php
//src/Acme/DemoBundle/Model

namespace AcmeDemoBundle\Model;

use Payum\Be2Bill\PaymentInstruction;

class Be2BillPaymentInstruction extends PaymentInstruction
{
    protected $id;
    
    public function getId()
    {
        return $this->id;
    }
}
```

and configure storage to use this model:

```yaml
#app/config/config.yml

payum:
    contexts:
        your_name_here:
            filesystem_storage:
                model_class: Acme\DemoBundle\Model\Be2BillPaymentInstruction
                storage_dir: %kernel.root_dir%/Resources/payments
                id_property: id
```

### Step 3. Capture payment: 

_**Note** : Here we assume you choose choose doctrine storage_  


```php
<?php
//src/Acme/DemoBundle/Controller
namespace AcmeDemoBundle\Controller;

use Symfony\Component\HttpFoundation\Request;
use Acme\DemoBundle\Entity\Be2billPaymentInstruction;

class PaymentController extends Controller 
{
    public function captureWithBe2BillAction(Request $request)
    {
        $contextName = 'your_context_name';
    
        $paymentContext = $this->get('payum')->getContext($contextName);
    
        /** @var PaypalPaymentInstruction */
        $instruction = $paymentContext->getStorage()->createModel();
        $instruction->setAmount(10005); //be2bill amount format is cents: for example:  100.05 (EUR). will be 10005.
        $instruction->setClientemail('user@email.com');
        $instruction->setClientuseragent($request->headers->get('User-Agent', 'Unknown'));
        $instruction->setClientip($request->getClientIp());
        $instruction->setClientident('payerId');
        $instruction->setDescription('Payment for digital stuff');
        $instruction->setOrderid('orderId');
        $instruction->setCardcode('5555 5567 7825 0000');
        $instruction->setCardcvv(123);
        $instruction->setCardfullname('John Doe');
        $instruction->setCardvaliditydate('15-11');
        
        return $this->forward('PayumBundle:Capture:do', array(
            'contextName' => $contextName,
            'model' => $instruction
        ));
    }
}
```

### Next Step

You are ready to read 

* [how to manage interactive](interactive_requests.md).
* [how to customize capture finished controller](customize_capture_finished_controller.md).
* [look at configuration reference](configuration_reference.md).