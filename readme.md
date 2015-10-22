# Implementace platební brány CSOB

## Klíče
1) Testovací prostředí
    - [CustomerTestKey] - testovací klíč zákazníka (*.pub)
    - [BankTestKey] - testovací klíč banky (*.pub)
2) Produkční prostředí
    - [CustomerKey] - klíč zákazníka
    - [BankKey] - klíč banky
    
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



[CustomerTestKey]: <https://github.com/joemccann/dillinger>
[BankTestKey]: <https://github.com/joemccann/dillinger>
[CustomerKey]: <https://github.com/joemccann/dillinger>
[BankKey]: <https://github.com/joemccann/dillinger.git>
[validace-implementace]: img/validace.png "Validace"

