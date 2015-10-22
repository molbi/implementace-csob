# Implementace platební brány CSOB

## Klíče
* Testovací prostředí
** [CustomerTestKey] - testovací klíč zákazníka (*.pub)
** [BankTestKey] - testovací klíč banky (*.pub)
* Produkční prostředí
** [CustomerKey] - klíč zákazníka
** [BankKey] - klíč banky
    
## PHP knihovna
Pro jendoduší volání API existuje PHP klihovna pro práci s CSOB platební bránou
-  https://github.com/ondrakoupil/csob

Návod jak implementova a použít knihovnu je velice dobře popsaný již na GitHubu knihovny, níže je ukázka implementace v projektu, kde jsem tuto knihovnu použil.

## Ukázka implementace



## Postup testování

Po implementaci je potřeba otestovat různé scénáře, vše je výbořně popsáno zde https://github.com/csob/paymentgateway/wiki/P%C5%99echod-do-produk%C4%8Dn%C3%ADho-prost%C5%99ed%C3%AD.
Stačí postupovat podle kroků, které jsou zde uvedeny. Různé scénáře se simulují pomocí různých typů karet https://github.com/csob/paymentgateway/wiki/Testovac%C3%AD-karty

Ve chvíli, kdy budete mít pocit, že jste vše ostestoval, příhlásíte se do administrace brány (testovací verze) a zažádáte o schválení implementace. Viz obrázek.
![Validace implementace][validace-implementace]



[CustomerTestKey]: <keys/test_rsa_M1MIPS0054.zip>
[BankTestKey]: <keys/mips_iplatebnibrana.csob.cz.zip>
[CustomerKey]: <asdf>
[BankKey]: <keys/mips_platebnibrana.csob.cz.zip>
[validace-implementace]: img/validace.png "Validace"

