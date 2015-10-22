# Implementace platební brány CSOB

## Klíče
1. Testovací prostředí
    
    [CustomerTestKey] - testovací klíč zákazníka (*.pub)
    
    [BankTestKey] - testovací klíč banky (*.pub)

2. Produkční prostředí
    
    [CustomerKey] - klíč zákazníka
    
    [BankKey] - klíč banky
    
## PHP knihovna

Pro jendoduší volání API existuje PHP klihovna pro práci s CSOB platební bránou
-  https://github.com/ondrakoupil/csob

Návod jak implementova a použít knihovnu je velice dobře popsaný již na GitHubu knihovny, níže je ukázka implementace v projektu, kde jsem tuto knihovnu použil.

## Ukázka implementace

Zde je ukázka třídy, kterou jsem pouzil ve svem projektu.

```php
<?php

namespace MCH\Services;

use Illuminate\Database\Eloquent\Model;
use OndraKoupil\Csob\Client, OndraKoupil\Csob\Config, OndraKoupil\Csob\Payment;

/**
 * Class PaymentGatewayService
 * @package MCH\Services
 */
class PaymentGatewayService
{

	/**
	 * @var
	 */
	private static $instance;

	/**
	 * @var Config null|Config
	 */
	private $_config = null;
	/**
	 * @var Client null
	 */
	private $_client = null;


	/**
	 * Init
	 */
	private function __construct()
	{
		$this->_setConfig();
		$this->_setClient();
	}

	/**
	 * @return PaymentGatewayService
	 */
	public static function getInstance()
	{
		if (is_null(self::$instance)) {
			self::$instance = new self();
		}
		return self::$instance;
	}


	/**
	 * @param Model $model
	 * @param array $params
	 */
	public function newPaymentProcess(Model $model, array $params = array(), array $item_params = array())
	{
		//vytvoření objektu platby pro CSOB
		/** @var Payment $payment */
		$payment = new Payment($model->id, null, $model->id);

		foreach($params as $param => $value) {
			$payment->$param = $value;
		}

		$totalCost = ($item_params['item_price'] + $item_params['shipping']) * 100;

		$payment->addCartItem($item_params['item_title'], 1, $totalCost );

		//ověření platby
		$this->_client->paymentInit($payment);

		//uložení payId do lokální platby
		$model->pay_id = $payment->getPayId();
		$model->save();

		//přesměrování na platební bránu
		$this->_client->redirectToGateway($payment);
		exit;
	}

	/**
	 * @return array|null
	 */
	public function getRetugningCustomer()
	{
		return $this->_client->receiveReturningCustomer();
	}

	/**
	 * @param $response
	 * @param Model $model
	 */
	public function paymentConfirmProcess($response, Model $model)
	{
		//nastavení stavu platby
		$this->_setPaymentState($response, $model);
		//odeslání emailu
		$this->_sendEmail($response);
	}

	/**
	 * @param Model $model
	 */
	public function reversePaymentProcess(Model $model)
	{
		$response = $this->_client->paymentReverse($model->pay_id);
		$this->_setPaymentState($response, $model);
	}

	/**
	 * @param Model $model
	 */
	public function refundPaymentProcess(Model $model)
	{
		$response = $this->_client->paymentRefund($model->pay_id);
		$this->_setPaymentState($response, $model);
	}

	/**
	 * @param $payId
	 * @return array|number
	 */
	public function checkState($payId)
	{
		return $this->_client->paymentStatus($payId);
	}

	/**
	 * Nastavení konfigurace pro komunikaci s CSOB
	 */
	private function _setConfig()
	{
		if (is_null($this->_config)) {
			$this->_config = new Config(
				env('CSOB_MERCHANT_ID'),
				base_path() . env('CSOB_MERCHANT_KEY'),
				base_path() . env('CSOB_KEY'),
				"MCH PHOTOGRAPHY",
				// Adresa, kam se mají zákazníci vracet poté, co zaplatí
				env('CSOB_RETURN_URL'),
				// URL adresa API - výchozí je adresa testovacího (integračního) prostředí,
				// až budete připraveni přepnout se na ostré rozhraní, sem zadáte
				// adresu ostrého API.
				env('CSOB_PB_URL')
			);
		}
	}

	/**
	 * Nastavení klienta pro komunikaci s CSOB
	 *
	 * @return Client
	 */
	private function _setClient()
	{
		if (is_null($this->_client)) {
			$this->_client = new Client($this->_config);
			try {
				$this->_client->testGetConnection();
				$this->_client->testPostConnection();
			} catch (Exception $e) {
				echo "Something went wrong: " . $e->getMessage();
			}
		}
	}

	/**
	 * Nastavneí stavu platby z návratového kodu
	 *
	 * @param $response
	 */
	private function _setPaymentState($response, Model $model)
	{
		switch ($response['paymentStatus']) {
			case 7:
				$model->status = 'paid';
				break;
			case 3:
				$model->status = 'cancelled';
				break;
			case 6:
				$model->status = 'expired';
				break;
			case 5:
				$model->status = 'stoped';
				break;
			case 4:
				$model->status = 'authorized';
				break;
			default:
				$model->status = $response['paymentStatus'];
				break;
		}
		$model->save();
	}

	/**
	 * Odeslání emailu v závislosti na stavu objednávky
	 *
	 * @param $response
	 */
	private function _sendEmail($response)
	{

	}

}
```


## Postup testování

Po implementaci je potřeba otestovat různé scénáře, vše je výbořně popsáno zde https://github.com/csob/paymentgateway/wiki/P%C5%99echod-do-produk%C4%8Dn%C3%ADho-prost%C5%99ed%C3%AD.
Stačí postupovat podle kroků, které jsou zde uvedeny. Různé scénáře se simulují pomocí různých typů karet https://github.com/csob/paymentgateway/wiki/Testovac%C3%AD-karty

Ve chvíli, kdy budete mít pocit, že jste vše ostestoval, příhlásíte se do administrace brány (testovací verze) a zažádáte o schválení implementace. Viz obrázek.

![Validace implementace][validace-implementace]



[CustomerTestKey]: <keys/test_rsa_M1MIPS0054.zip>
[BankTestKey]: <keys/mips_iplatebnibrana.csob.cz.zip>
[BankKey]: <keys/mips_platebnibrana.csob.cz.zip>
[validace-implementace]: img/validace.png "Validace"

