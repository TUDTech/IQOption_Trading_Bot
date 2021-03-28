# IQ Option API

---

## Sobre a API

Desenvolvida em Python 3.7, você pode estar utilizando em "alto nivel" ou "baixo nivel", esta API pode trabalhar com Optções & Digital & Forex & Ações & Commodities & Crypto & ETFs;

```Python
# Alto Nivel
from iqoptionapi.stable_api import IQ_Option

# Baixo Nivel
from iqoptionapi.api import IQOptionAPI
```

```bash
.
├── docs
├── iqoptionapi(Código da API)
    ├── http(Realiza requisições HTTP GET/POST)
    └── ws
        ├── chanels(Utiliza o WebSocket)
        └── objects(Retorna as informações do WebSocket)
```

---

## Instalação e Atualização

Para Python 3

```bash
pip3 install -U git+git://github.com/Lu-Yi-Hsun/iqoptionapi.git
```
Para Python 2

```bash
pip2 install -U git+git://github.com/Lu-Yi-Hsun/iqoptionapi.git
```

---

## Funções e exemplos

### Importar o projeto para seu código

```python
from iqoptionapi.stable_api import IQ_Option
```

---

## Soluções de problemas/erros

### Minha conta tem verificação em duas etapas, como proceder?

Infelizmente a API não consegue trabalhar com contas que possuem verificação em duas etapas, será necessário você 
desativar a verificação em duas etapas para poder utilizar a APO

### Erros com WebSocket ou ao logar

Caso ocorra erros com websocket, o mesmo deve ser desinstalado e reinstalado(0.56)!

```bash
pip uninstall websocket-client
pip install websocket-client==0.56
```

### Conflito entre WebSocket e WebSocket-Cliente

Este erro pode vir a acontecer caso você esteja usando Anaconda, por exemplo

```bash
pip uninstall websocket
pip install websocket-client==0.56.0
```

---

## Debug 

Ligado

```python
import logging
logging.basicConfig(level=logging.DEBUG,format='%(asctime)s %(message)s')
```

Desligado

```python
import logging
logging.disable(level=(logging.DEBUG))
```

---

## Como realizar login

```python
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email", "senha")
```

### Limitar tentativas de reconexão com set_max_reconnect()

O valor padrão é 5, para evitar que sua conta e IP sejam banidos pela IQ por exceder o limite máximo de tentativas de conexão.

```python
API.set_max_reconnect(number)
```

### Reconectar e checar se está conectado

Caso ocorra algum erro e a conexão com a IQ seja perdida, você pode estar implementando isto

```python
from iqoptionapi.stable_api import IQ_Option
import time


API = IQ_Option("email", "password")
API.set_max_reconnect(-1) # Valor negativo irá deixar o numero máximo de reconexões infinito(cuidado)!

while True:
	if API.check_connect() == False: # Detecta se o WebSocket está conectado ou não
		print("Conexao perdida, tentando reconectar..")
        API.connect() # Realizando a conexão novamente
        print("Reconectado com sucesso!")
    time.sleep(1)
```
 
### Checar conexão

Função retorna True ou False

```python
print(API.check_connect())
```

### Conectar(ou reconectar)

```python
API.connect()
```

---

## Tipo de conta e banca

### Retornar sua banca com get_balance()

```python
API.get_balance()
```

### Resetar conta de TREINAMENTO (10k)

Função para resetar a conta de treinamento(depositar os 10k de testes)

```python
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email","password")

print(API.reset_practice_balance())
```

### Alterar entre conta REAL e de TREINAMENTO

```python
API.change_balance(TIPO)
		#TIPO: "PRACTICE" ou "REAL"
```

---

## Retornar ativos e verificar se estão aberto

ATENÇÃO: Tome cuidado, get_all_open_time() é pesado para a internet

- Função get_all_open_time() retorna um DICT
- "cfd" inclue ações,Commodities e ativos de ETFs

Verificar se esta aberto
DICT["forex"/"cfd"/"crypto"/"digital"/"turbo"/"binary"][NOME DO ATIVO]["open"]
> O retorno é em True ou False

```python
from iqoptionapi.stable_api import IQ_Option
API = IQ_Option("email", "password")

ATIVOS = API.get_all_open_time()

#Checando se está aberto ou não
print(ATIVOS["forex"]["EURUSD"]["open"]) 
print(ATIVOS["cfd"]["FACEBOOK"]["open"]) #Ações,Commodities e ETFs
print(ATIVOS["crypto"]["BTCUSD-L"]["open"])
print(ATIVOS["digital"]["EURUSD-OTC"]["open"])

#Binarias tem dois modos diferentes: "turbo" e "binary"
print(ALL_Asset["turbo"]["EURUSD-OTC"]["open"])
print(ALL_Asset["binary"]["EURUSD-OTC"]["open"])
```

Caso você indicar um ativo inexistente, irá ser retornado apenas "{}" ou None

Para exibir todas os ativos
 
```python
from iqoptionapi.stable_api import IQ_Option
API = IQ_Option("email", "password")

ATIVOS=API.get_all_open_time()

for tipo, data in ATIVOS.items():
    for ativo_nome,value in data.items():
        print(tipo,ativo_nome,value["open"])
```
---

### Ver o nome e ID de todos os ativos
- [Arquivo com lista de ativos e id's](iqoptionapi/constants.py)

```python
print(API.get_all_ACTIVES_OPCODE())
```

---

### get_async_order()
Pegar informações sobre ordem/operação pelo ID

```python
from iqoptionapi.stable_api import IQ_Option
import time

API = IQ_Option("email", "password")
 
PARIDADE = "EURUSD"
duracao = 1 #Tempo em minutos: 1 ou 5
entrada = 1
direcao = "call" #call ou put

print("__Para_Opções_Binarias__")
_,id = API.buy(entrada, PARIDADE, direcao, duracao)
while API.get_async_order(id)==None:
    pass
print(API.get_async_order(id))
print("\n\n")


print("__Para_Opcoes_Digitais__")
id = API.buy_digital_spot(PARIDADE, entrada, direcao, duracao)
while API.get_async_order(id)==None:
    pass
order_data = API.get_async_order(id)
print(API.get_async_order(id))
print("\n\n")
```

---

### "Humor dos Traders"

Por enquanto, só está disponivel para **binario**

Exemplo
```python
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email", "password")

Paridade = "EURUSD"

API.start_mood_stream(Paridade)
print(API.get_traders_mood(Paridade))
API.stop_mood_stream(Paridade)
```

#### get_traders_mood()

Exibir/pegar a porcentagem de **call**, se você quiser saber a porcentagem de put, basta fazer 100-humor_call

```python
API.get_traders_mood(Paridade)
	# Retorno: Sera do tipo float que representa em porcentagem os 'calls'
	# Se você quiser saber a porcentagem de put, tente 100-API.get_traders_mood(Paridade)
```

#### get_all_traders_mood()
Pega tudo
```python
API.get_all_traders_mood(Paridade)
	# Retorno: (dict) com todos os 'humores'
```

---

### Para Opções (binarias)

#### Realizar operação na binaria com buy()

Exemplo
```python
from iqoptionapi.stable_api import IQ_Option
import time

API = IQ_Option("email", "pass")

Entrada = 1
Paridade = "EURUSD"
Direcao = "call" # Ou "put"
Duracao = 1

id = API.buy(Entrada, Paridade, direcao, Duracao)
```

Explicação
```python
API.buy(Entrada, Paridade, direcao, duracao)
                # Entrada: Quanto em reais/dolares você quer usar na entrada (Minimo 1 dolar ou 2 reais), a informação tem que ser do tipo int ou float
                # Paridade: Qual paridade você deseja operar
                # Direcao: "call"/"put", informação deve ser do tipo str
                # Duracao: Qual o tempo da operação em minutos, cuidado com operações muito longas para não dar erro(mercado fechado)!
                # dados retornados: (None ou id_number):if sucess return (id_number) esle return(None) 2.1.5 change this 
```

---

#### Operações simultaneas com buy_multi()

Exemplo
```python
from iqoptionapi.stable_api import IQ_Option
API = IQ_Option("email", "password")

Entrada = []
Paridade = []
Direcao = []
Duracao = []

Entrada.append(1)
Paridade.append("EURUSD")
Direcao.append("call") # ou put
Duracao.append(1)

Entrada.append(1)
Paridade.append("EURAUD")
Direcao.append("call")#put
Duracao.append(1)

print("buy_multi()")
ids = API.buy_multi(Entrada, Paridade, Direcao, Duracao)

print("Checar resultado em apenas 1 paridade (ids[0])")
print(API.check_win_v3(ids[0]))
```

---

#### Tempo restante para operação com get_remaning()

Formula: tempo de compra = tempo restante - 30

```python
from iqoptionapi.stable_api import IQ_Option
API=IQ_Option("email", "password")

Entrada = 1
Paridade = "EURUSD"
Direcao = "call"#or "put"
Duracao = 1

while True:
    tempo_restante = API.get_remaning(Duracao)
    tempo_de_compra = tempo_restante-30
    if tempo_de_compra < 4: #Compre Binaria em tempo_de_compra < 4
        API.buy(Entrada, Paridade, Direcao, Duracao)
        break
```

#### Vender operação com sell_option()
O ID('s) passados para o sell_option() devem ser int ou um list contendo os id's
```python
API.sell_option(sell_ids)
```

Exemplo
```python
from iqoptionapi.stable_api import IQ_Option
import time

API = IQ_Option("email", "password")

Entrada = 1
Paridade = "EURUSD"
Direcao = "call"
Duracao = 1

id = API.buy(Entrada, Paridade, Direcao, Duracao)
id2 = API.buy(Entrada, Paridade, Direcao, Duracao)

time.sleep(5)

sell_all=[]
sell_all.append(id)
sell_all.append(id2)

print(API.sell_option(sell_all))
```

#### Verificar resultado da operação nas **BINÁRIA**

> As funções check_win() e check_win_v2() pararam de funcionar

Para pegarmos o resultado de uma operação feito na binarias, podemos estar utilizando o check_win_v3() ou o check_win_v4()


###### check_win_v3()
```python
from iqoptionapi.stable_api import IQ_Option
import time

API = IQ_Option("email", "password")

Entrada = 1
Paridade = "EURUSD"
Direcao = "call"
Duracao = 1

id = API.buy(Entrada, Paridade, Direcao, Duracao)

time.sleep(5)

print(API.check_win_v3(id))
		# Você precisa ter o id da operação para poder fazer a verificação
		# check_win_v3() não é aconselhado para pegar resultado de operação realizada a muitos trades atras, use o check_win_v4() nesta ocasião
		# Esta função irá rodar em looping até retornar o resultado tipo 'tuple'
		# Resultado retornado no padrão 'True/False',lucro
```

###### check_win_v4()
```python
from iqoptionapi.stable_api import IQ_Option
import time

API = IQ_Option("email", "password")

Entrada = 1
Paridade = "EURUSD"
Direcao = "call"
Duracao = 1

id = API.buy(Entrada, Paridade, Direcao, Duracao)

time.sleep(5)

print(API.check_win_v4(id))
		# Você precisa ter o id da operação para poder fazer a verificação
		# Esta função irá rodar em looping até retornar o resultado tipo 'tuple'
		# Resultado retornado no padrão 'True/False',lucro
```


---

### Dados brutos da **BINÁRIA**

#### get_all_init()

"get_binary_option_detail()" e "get_all_profit()" são baseados no "get_all_init()", para retornar os dados "brutos", você pode utilizar:

Exemplo
```python
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email", "password")

print(API.get_all_init())
```

![](image/expiration_time.png)

#### get_binary_option_detail()

Exemplo
```python
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email", "password")

d = API.get_binary_option_detail()

print(d["CADCHF"]["turbo"])
print(d["CADCHF"]["binary"])
```

#### get_all_profit()

Exemplo
```python
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email", "password")
d = API.get_all_profit()

print(d["CADCHF"]["turbo"])
print(d["CADCHF"]["binary"])
```
---
#### Pegar histórico de trading das **BINÁRIAS**


Temos dois modos para fazer isto, para ambos precisamos indicar quantos 'trades' você quer retornar do histórico de trading ( apenas das binárias )

###### get_optioninfo()
```python
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email", "password")

print(API.get_optioninfo(10))
```

###### get_optioninfo_v2()

```python
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email", "password")

print(API.get_optioninfo_v2(10))

```


#### Pegar opções feitas por outro dispositivo com get_option_open_by_other_pc()
Se sua conta está logada em outro celular/PC e está realizando operações, você pode "pegar" a operação do modo abaixo

```python
import time
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email", "password")

while True:
    # Abra o Website da IQ Option e faça alguma operação por lá
    if API.get_option_open_by_other_pc()!={}:
        break
    time.sleep(1)
	
print("Pegar operação feita nesta conta por outro dispositivo")
print(API.get_option_open_by_other_pc())

id=list(API.get_option_open_by_other_pc().keys())[0]
API.del_option_open_by_other_pc(id)

print("Depois de deleter com del_option_open_by_other_pc(), executamos get_option_open_by_other_pc() novamente ")
print(API.get_option_open_by_other_pc())
```

---
---

### Para digitais


#### get_all_strike_list_data()

Formato da informação retornada

```python
{'1.127100': {  'call': {'profit': None, 'id': 'doEURUSD201811120649PT1MC11271'},   'put': {'profit': 566.6666666666666, 'id': 'doEURUSD201811120649PT1MP11271'}	}.......}  
```

Exemplo de uso
```python
from iqoptionapi.stable_api import IQ_Option
import time

API = IQ_Option("email", "password")

Paridade = "EURUSD"
Duracao = 1 # 1 ou 5 minutos

API.subscribe_strike_list(Paridade, Duracao)

while True:
    data = API.get_realtime_strike_list(Paridade, Duracao)
    for preco in data:
        print("Preco",preco,data[preco])
    time.sleep(5)
	
API.unsubscribe_strike_list(Paridade, Duracao)
```

#### Realizar operações nas Digitais com buy_digital_spot()

Abrir operação na digital com preço atual
```python
from iqoptionapi.stable_api import IQ_Option
 
API=IQ_Option("email","password")

Paridade = "EURUSD"
Duracao = 1#minute 1 or 5
Entrada = 1
Direcao = "call"

print(API.buy_digital_spot(Paridade, Entrada, Direcao, Duracao))
```

#### Realizar operações nas Digitais com buy_digital()
Para poder utilizar esta função, você devera pegar o instument_id pelo API.get_realtime_strike_list()

```python
buy_check,id=API.buy_digital(Entrada, instrument_id)
```

#### Pegar lucro pós venda com get_digital_spot_profit_after_sale()

![](image/profit_after_sale.png)

Exemplo
```python
from iqoptionapi.stable_api import IQ_Option 

API = IQ_Option("email", "passord")

Paridade = "EURUSD"
Duracao = 1 #1 ou 5 minutos
Entrada = 100
Direcao = "put"
 
API.subscribe_strike_list(Paridade,Duracao)

id=API.buy_digital_spot(Paridade,Entrada,Direcao,Duracao) 
 
while True:
    PL=API.get_digital_spot_profit_after_sale(id)
    if PL!=None:
        print(PL)
     
```

#### Pegar payout com get_digital_current_profit()
Esta função funciona somente para digitais!

Exemplo
```python
from iqoptionapi.stable_api import IQ_Option
import time

API = IQ_Option("email", "password")

Paridade = "EURUSD"
Duracao = 1#minute 1 or 5

API.subscribe_strike_list(Paridade, Duracao)
while True:
    data=API.get_digital_current_profit(Paridade, Duracao)
    print(data) # Nos primeiros retornos pode vir "False", aguarde alguns segundos que começara a retornar os valores do tipo float
    time.sleep(1)	
API.unsubscribe_strike_list(Paridade, Duracao)
```


#### Verificar resultado da operação nas **DIGITAIS**

Para pegarmos o resultado de uma operação feito nas digitais, podemos estar utilizando o check_win_digital() ou o check_win_digital_v2()


###### check_win_digital()
Esta função foi implementada com get_digital_position()
```python
API.check_win_digital(id) # get the id from API.buy_digital
	# Retorno tipo tuple: status_operacao,lucro
	# if you loose:Ture,o
	# if you win:True,1232.3
	# if trade not clode yet:False,None
```


###### check_win_digital_v2()
Função utilizada para retornar se o resultado de uma operação nas digitais

```python
API.check_win_digital_v2(id)#get the id from API.buy_digital
	# Retorno tipo tuple: status_operacao,lucro
	# Se você perder: True,0
	# Se você ganhar: True,1232.3 
	# Se a operação não encerrou ainda: False,None
```

Exemplo
```python
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email", "password")


Paridade = "EURUSD"
Duracao = 1 #1 ou 5 minutos
Entrada = 1
Direcao = "call"

id= API.buy_digital_spot(Paridade, Duracao, Direcao, Entrada)
print(id)

if id != "error":
    while True:
        status,lucro = API.check_win_digital_v2(id)
        if status == True:
            break
    if lucro < 0:
        print("Voce perdeu "+str(win)+"$")
    else:
        print("Voce ganhou "+str(win)+"$")
else:
    print("Por favor, tente novamente")
```


#### Vender/fechar operação nas digitais com close_digital_option()
```python
API.close_digital_option(id)
```


#### Pegar informações das **DIGITAIS**


Utilizando get_digital_position()
```python
from iqoptionapi.stable_api import IQ_Option

API = IQ_Option("email","password")

Paridade = "EURUSD-OTC"
Duracao = 1#minute 1 or 5
Entrada = 1
Direcao = "call"#put


 
id = API.buy_digital_spot(Paridade, Entrada, Direcao, Duracao) 

while True:
    check,_= API.check_win_digital(id)
    if check:
        break
print(API.get_digital_position(id))
print(API.check_win_digital(id))
```
Utilizando funções get_positions(), get_digital_position() e get_position_history()
```python
#print(API.get_order(id)) # Não funciona com digitais

print(API.get_positions("digital-option"))
print(API.get_digital_position(2323433))#Deve ser colocado o ID
print(API.get_position_history("digital-option"))
```

---

### <a id=forex>For Forex&Stock&Commodities&Crypto&ETFs</a>

#### you need to check Asset is open or close!

try this api [get_all_open_time](#checkopen)
![](image/asset_close.png)



#### <a id=instrumenttypeid>About instrument_type and instrument_id</a>

you can search instrument_type and instrument_id from this file

[search instrument_type and instrument_id](instrument.txt)
 

#### Sample
```python
from iqoptionapi.stable_api import IQ_Option
API=IQ_Option("email","password")

instrument_type="crypto"
instrument_id="BTCUSD"
side="buy"#input:"buy"/"sell"
amount=1.23#input how many Amount you want to play

#"leverage"="Multiplier"
leverage=3#you can get more information in get_available_leverages()

type="market"#input:"market"/"limit"/"stop"

#for type="limit"/"stop"

# only working by set type="limit"
limit_price=None#input:None/value(float/int)

# only working by set type="stop"
stop_price=None#input:None/value(float/int)

#"percent"=Profit Percentage
#"price"=Asset Price
#"diff"=Profit in Money

stop_lose_kind="percent"#input:None/"price"/"diff"/"percent"
stop_lose_value=95#input:None/value(float/int)

take_profit_kind=None#input:None/"price"/"diff"/"percent"
take_profit_value=None#input:None/value(float/int)

#"use_trail_stop"="Trailing Stop"
use_trail_stop=True#True/False

#"auto_margin_call"="Use Balance to Keep Position Open"
auto_margin_call=False#True/False
#if you want "take_profit_kind"&
#            "take_profit_value"&
#            "stop_lose_kind"&
#            "stop_lose_value" all being "Not Set","auto_margin_call" need to set:True

use_token_for_commission=False#True/False

check,order_id=API.buy_order(instrument_type=instrument_type, instrument_id=instrument_id,
            side=side, amount=amount,leverage=leverage,
            type=type,limit_price=limit_price, stop_price=stop_price,
            stop_lose_value=stop_lose_value, stop_lose_kind=stop_lose_kind,
            take_profit_value=take_profit_value, take_profit_kind=take_profit_kind,
            use_trail_stop=use_trail_stop, auto_margin_call=auto_margin_call,
            use_token_for_commission=use_token_for_commission)
print(API.get_order(order_id)) 
print(API.get_positions("crypto"))
print(API.get_position_history("crypto"))
print(API.get_available_leverages("crypto","BTCUSD"))
print(API.close_position(order_id))
print(API.get_overnight_fee("crypto","BTCUSD"))
```
 



#### Buy

return (True/False,buy_order_id/False)

if Buy sucess return (True,buy_order_id)

"percent"=Profit Percentage

"price"=Asset Price

"diff"=Profit in Money

|parameter|||||
--|--|--|--|--|
instrument_type|[instrument_type](#instrumenttypeid)
instrument_id| [instrument_id](#instrumenttypeid)
side|"buy"|"sell"
amount|value(float/int)
leverage|value(int)
type|"market"|"limit"|"stop"
limit_price|None|value(float/int):Only working by set type="limit"
stop_price|None|value(float/int):Only working by set type="stop"
stop_lose_kind|None|"price"|"diff"|"percent"
stop_lose_value|None|value(float/int)
take_profit_kind|None|"price"|"diff"|"percent"
take_profit_value|None|value(float/int)
use_trail_stop|True|False
auto_margin_call|True|False
use_token_for_commission|True|False

```python
check,order_id=API.buy_order(
            instrument_type=instrument_type, instrument_id=instrument_id,
            side=side, amount=amount,leverage=leverage,
            type=type,limit_price=limit_price, stop_price=stop_price,
            stop_lose_kind=stop_lose_kind,
            stop_lose_value=stop_lose_value,
            take_profit_kind=take_profit_kind,
            take_profit_value=take_profit_value,
            use_trail_stop=use_trail_stop, auto_margin_call=auto_margin_call,
            use_token_for_commission=use_token_for_commission)

```
#### <a id=changeorder>change_order</a>

##### change PENDING
![](image/change_ID_Name_order_id.png)

##### change Position
![](image/change_ID_Name_position_id.png)

|parameter|||||
--|--|--|--|--|
ID_Name|"position_id"|"order_id"
order_id|"you need to get order_id from buy_order()"
stop_lose_kind|None|"price"|"diff"|"percent"
stop_lose_value|None|value(float/int)
take_profit_kind|None|"price"|"diff"|"percent"
take_profit_value|None|value(float/int)
use_trail_stop|True|False
auto_margin_call|True|False


##### sample
```python
ID_Name="order_id"#"position_id"/"order_id"
stop_lose_kind=None
stop_lose_value=None
take_profit_kind="percent"
take_profit_value=200
use_trail_stop=False
auto_margin_call=True
API.change_order(ID_Name=ID_Name,order_id=order_id,
                stop_lose_kind=stop_lose_kind,stop_lose_value=stop_lose_value,
                take_profit_kind=take_profit_kind,take_profit_value=take_profit_value,
                use_trail_stop=use_trail_stop,auto_margin_call=auto_margin_call)
```

---


#### get_order

 
get infomation about buy_order_id

return (True/False,get_order,None)

```python
API.get_order(buy_order_id)
```

#### get_pending
you will get there data

![](image/get_pending.png)

```python
API.get_pending(instrument_type)
```
#### get_positions

you will get there data

![](image/get_positions.png)

return (True/False,get_positions,None)


:exclamation: not support ""turbo-option""

instrument_type="crypto","forex","fx-option","multi-option","cfd","digital-option"

```python
API.get_positions(instrument_type)
```

#### get_position
you will get there data

![](image/get_position.png)

you will get one position by buy_order_id

return (True/False,position data,None)

```python
API.get_positions(buy_order_id)
```

#### get_position_history

you will get there data

![](image/get_position_history.png)

return (True/False,position_history,None)

```python
API.get_position_history(instrument_type)
```
#### <a id=getpositionhistoryv2>get_position_history_v2</a>

instrument_type="crypto","forex","fx-option","turbo-option","multi-option","cfd","digital-option"

get_position_history_v2(instrument_type,limit,offset,start,end)

```python
from iqoptionapi.stable_api import IQ_Option
import logging
import random
import time
import datetime
logging.basicConfig(level=logging.DEBUG,format='%(asctime)s %(message)s')
API=IQ_Option("email","password")

#instrument_type="crypto","forex","fx-option","turbo-option","multi-option","cfd","digital-option"  
instrument_type="digital-option"
limit=2#How many you want to get
offset=0#offset from end time,if end time is 0,it mean get the data from now 
start=0#start time Timestamp
end=0#Timestamp
data=API.get_position_history_v2(instrument_type,limit,offset,start,end)

print(data)

#--------- this will get data start from 2019/7/1(end) to 2019/1/1(start) and only get 2(limit) data and offset is 0
instrument_type="digital-option"
limit=2#How many you want to get
offset=0#offset from end time,if end time is 0,it mean get the data from now 
start=int(time.mktime(datetime.datetime.strptime("2019/1/1", "%Y/%m/%d").timetuple()))
end=int(time.mktime(datetime.datetime.strptime("2019/7/1", "%Y/%m/%d").timetuple()))
data=API.get_position_history_v2(instrument_type,limit,offset,start,end)
print(data)

```

#### get_available_leverages

get available leverages

return (True/False,available_leverages,None)

```python
API.get_available_leverages(instrument_type,actives)
```
#### cancel_order

you will do this

![](image/cancel_order.png)

return (True/False)

```python
API.cancel_order(buy_order_id)
```

#### close_position

you will do this

![](image/close_position.png)

return (True/False)

```python
API.close_position(buy_order_id)
```

#### get_overnight_fee

return (True/False,overnight_fee,None)

```python
API.get_overnight_fee(instrument_type,active)
```
---
---

### Candle

#### get candles
:exclamation:

 get_candles can not get "real time data" ,it will late about 30sec

if you very care about real time you need use 

"get realtime candles" OR "collect realtime candles"

sample 

""now"" time 1:30:45sec

1.  you want to get  candles 1:30:45sec now
    
    you may get 1:30:15sec data have been late approximately 30sec

2.  you want to get  candles 1:00:33sec 

    you will get the right data

```python
API.get_candles(ACTIVES,interval,count,endtime)
            #ACTIVES:sample input "EURUSD" OR "EURGBP".... youcan
            #interval:duration of candles
            #count:how many candles you want to get from now to past
            #endtime:get candles from past to "endtime"
```
:exclamation:
try this code to get more than 1000 candle
```python
from iqoptionapi.stable_api import IQ_Option
import time
API=IQ_Option("email","password")
end_from_time=time.time()
ANS=[]
for i in range(70):
    data=API.get_candles("EURUSD", 60, 1000, end_from_time)
    ANS =data+ANS
    end_from_time=int(data[0]["from"])-1
print(ANS)
```

#### get realtime candles

##### Sample 
```python
from iqoptionapi.stable_api import IQ_Option
import logging
import time
#logging.basicConfig(level=logging.DEBUG,format='%(asctime)s %(message)s')
print("login...")
API=IQ_Option("email","password")
goal="EURUSD"
size="all"#size=[1,5,10,15,30,60,120,300,600,900,1800,3600,7200,14400,28800,43200,86400,604800,2592000,"all"]
maxdict=10
print("start stream...")
API.start_candles_stream(goal,size,maxdict)
#DO something
print("Do something...")
time.sleep(10)

print("print candles")
cc=API.get_realtime_candles(goal,size)
for k in cc:
    print(goal,"size",k,cc[k])
print("stop candle")
API.stop_candles_stream(goal,size)
```
 
##### start_candles_stream
 
* input:
    * goal:"EURUSD"...
    * size:[1,5,10,15,30,60,120,300,600,900,1800,3600,7200,14400,28800,43200,86400,604800,2592000,"all"]
    * maxdict:set max buffer you want to save

size

![](image/time_interval.png)

##### get_realtime_candles
* input:
    * goal:"EURUSD"...
    * size:[1,5,10,15,30,60,120,300,600,900,1800,3600,7200,14400,28800,43200,86400,604800,2592000,"all"]
* output:
    * dict
##### stop_candles_stream
* input:
    * goal:"EURUSD"...
    * size:[1,5,10,15,30,60,120,300,600,900,1800,3600,7200,14400,28800,43200,86400,604800,2592000,"all"]

---
### time

#### <a id=timestamp> get_server_timestamp</a>
the get_server_timestamp time is sync with iqoption
```python
API.get_server_timestamp()
```

#### <a id=purchase>Purchase Time</a>
this sample get the Purchase time clock
```python
import time

#get the end of the timestamp by expiration time
def get_expiration_time(t):
    exp=time.time()#or API.get_server_timestamp() to get more Precision
    if (exp % 60) > 30:
        end = exp - (exp % 60) + 60*(t+1)
    else:
        end = exp - (exp % 60)+60*(t)
    return end
    
expiration_time=2

end_time=0
while True:
    if end_time-time.time()-30<=0:
        end_time = get_expiration_time(expiration_time)
    print(end_time-time.time()-30)
    time.sleep(1)
```
---
### Get top_assets_updated

instrument_type="binary-option"/"digital-option"/"forex"/"cfd"/"crypto"

```python
from iqoptionapi.stable_api import IQ_Option
import logging
import time
#logging.basicConfig(level=logging.DEBUG,format='%(asctime)s %(message)s')
API=IQ_Option("email","password")
instrument_type="digital-option"#"binary-option"/"digital-option"/"forex"/"cfd"/"crypto"
API.subscribe_top_assets_updated(instrument_type)

print("__Please_wait_for_sec__")
while True:
    if API.get_top_assets_updated(instrument_type)!=None:
        print(API.get_top_assets_updated(instrument_type))
        print("\n\n")
    time.sleep(1)
API.unsubscribe_top_assets_updated(instrument_type)
```

#### get popularity by top_assets_updated() api

https://github.com/Lu-Yi-Hsun/iqoptionapi/issues/131

![](https://user-images.githubusercontent.com/7738916/66943816-c9ee1380-f000-11e9-996e-e06efba64101.png)

```python
from iqoptionapi.stable_api import IQ_Option
import logging
import time
import operator
 
#logging.basicConfig(level=logging.DEBUG,format='%(asctime)s %(message)s')
def opcode_to_name(opcode_data,opcode):
    return list(opcode_data.keys())[list(opcode_data.values()).index(opcode)]            

API=IQ_Option("email","password")
API.update_ACTIVES_OPCODE()
opcode_data=API.get_all_ACTIVES_OPCODE()

instrument_type="digital-option"#"binary-option"/"digital-option"/"forex"/"cfd"/"crypto"
API.subscribe_top_assets_updated(instrument_type)


print("__Please_wait_for_sec__")
while True:
    if API.get_top_assets_updated(instrument_type)!=None:
        break

top_assets=API.get_top_assets_updated(instrument_type)
popularity={}
for asset in top_assets:
    opcode=asset["active_id"]
    popularity_value=asset["popularity"]["value"]
    try:
        name=opcode_to_name(opcode_data,opcode)
        popularity[name]=popularity_value
    except:
        pass
 
 
sorted_popularity = sorted(popularity.items(), key=operator.itemgetter(1))
print("__Popularity_min_to_max__")
for lis in sorted_popularity:
    print(lis)

API.unsubscribe_top_assets_updated(instrument_type)
```
