# Tutorial-Bluetooth-HC-05
Tutorial para a comunicação Bluetooth através do módulo HC-05 entre duas Placas NUCLEO-F401RE
# Introdução:
Este tutorial tem o objetivo de introduzir o leitor à utilização do módulo Bluetooth HC-05 para a comunicação entre duas placas NUCLEO-F401RE. O tutorial será dividido em três partes. A primeira parte será dedicada a entender como conectar dois módulos entre si. A segunda parte será uma prática com LED, na qual será ensinado como realizar as conexões físicas e como a informação é transmitida entre os módulos. Por fim, serão apresentadas funções interessantes para projetos mais complexos.
# Começando com o HC-05
O módulo Bluetooth HC-05 é um transmissor de informações. Para que dois módulos consigam se comunicar entre si, é necessário que um dos módulos seja o mestre e o outro, o escravo. O mestre, sempre que ligado, irá procurar pelo escravo, e vice-versa. Para que isso ocorra, utilizamos a placa FT232RL USB to TTL, que, juntamente com o Termite, permite que os dois módulos se reconheçam e interajam entre si. 
Abaixo está um passo a passo para auxiliar na configuração dos módulos. Em "Referências" estão dois links que também falam sobre esse assunto.



![image](https://github.com/user-attachments/assets/a65120a2-5fe1-4ad7-94e0-9a7bc2eb8a8f)

## Passo a passo para a configuração dos módulos

- Baixar o Termite que irá permitir a configuração dos módulos (link para [download](https://www.compuphase.com/software_termite.htm))
- Conectar o primeiro módulo com o FT232RL como mostrado na imagem a seguir:
- Importante! ao conectar o módulo no FT232RL, mantenha pressionado o botão presente no módulo bluetooth para que ele entre em modo de configuração. 
  
![image](https://github.com/user-attachments/assets/0a3bebd3-21e9-4c20-abcd-2623fb1a1b99)

- Abrir o terminal do termite e escrever "AT" que serve como teste para ver se a conexão funcionou, ele deve devolver um "OK".
- Agora que sabemos que a conexão funcionou escreva “AT+ADDR?” que vai devolver o endereço do módulo. Anote o endereço. Você deve receber algo do tipo: 98d3:34:905d3f
- Por último escreva “AT+ROLE=0” que vai configurar o módulo como slave. Desconecte o módulo bluetooth do FT232RL e conecte o outro módulo.
- Escreva "AT" para testar a conexão. Após receber o "OK", escreva “AT+ROLE=1” que vai configurar o módulo como master.
- Escreva “AT+CMODE=0” que vai configurar o módulo para o modo de conexão de "Endereço fixo".
- Por último escreva “AT+BIND= endereço que você anotou preveamente” para que o módulo mestre procure pelo servo para se conectar. Importante! Ao escrever o endereço mude os ":" por ",". Exemplo: “AT+BIND=98d3,34,905d3f".
  


Agora que a configuração esta feita, podemos partir para a primeira prática e ver os módulos funcionando!
# Prática: LED
Antes de tudo, é preciso realizar as conexões corretas entre o módulo e a placa núcleo. O VCC do módulo pode ser alimentado pelo 3.3V da placa, o GND do módulo no GND da placa, o RXD é o receptor de informações e na placa núcleo é o pino PA_10 e o TXD que é o transmissor é o pino PA_9. A foto abaixo mostra o mapa de pinos da NUCLEO-401RE para auxiliar-los nas conexões.

![image](https://github.com/user-attachments/assets/df690810-7d15-4b23-81ac-c86b935ecd67)


![image](https://github.com/user-attachments/assets/2424c8d0-1b5f-4a13-8020-2011685e0021)

Agora podemos rodar o primeiro código utilizando os módulos como comunicação entre as placas. O objetivo é pressionar o botão do usuário em uma placa e acender o LED da outra placa, sem conexões físicas!
Para esse objetivo uma placa vai apenas enviar informação (botão precionado ou não) e a outra placa vai apenas receber e executar uma rotina (precionado liga o LED). Então para começar vamos ver como fica o código para enviar informação.

```C++
#include "mbed.h"
DigitalIn botao(USER_BUTTON); // Botão do usuário
Serial bt (PA_9, PA_10); // Define as portas do módulo bluetooth
int main() {
  while(1) {
    if(botao == 1) { // se o botão for precionado enviar 'l'
        bt.putc('l');
        wait_ms(200);
    }
    if(botao == 0) { // se o botão não for precionado enviar 'd'
        bt.putc('d');
        wait_ms(200);
    }
  }
}
```
O código acima envia informações diferentes para a outra placa caso o botão do usuário seja apertado ou não. Para enviar uma informação utilizamos o bt.putc().

```C++
#include "mbed.h"
DigitalOut myled(LED1); // LED da núcleo
Serial bt (PA_9, PA_10); // Define as portas do módulo bluetooth
int main() {
  while(1) {
    char c = bt.getc(); // Atribui a informação recebida pelo módulo à char "c"
    if(c == 'l') { // se a informaçã recebida for 'c' ligar o LED
        myled = 1;
        wait_ms(200);
    }
    if(c == 'd') { // se a informaçã recebida for 'd' desligar o LED
        myled = 0;
        wait_ms(200);
    }
  }
}
```
O código acima indentifica as informações recebidas pela outra placa e acende ou não o LED dependendo do conteúdo da char 'c'. Para receber uma informação utilizamos o bt.getc().
# Funções interessantes
Com o que já foi apresentado é possível implementar diversas rotinas. Porém, existem outras funções e estratégias para deixar o código mais eficiente e explorar outras funções do módulo Bluetooth.
## Callback
A ideia da função de callback é sempre receber a informação do módulo bluetooth e guarda-la em uma variável, assim, no código a variável que guarda a informação do bluetooth sempre estará atualizada com a nova informação. O intuito é que essa função seja a primeira, ou uma das primeiras, linhas de código dentro da "main".
```C++
#include "mbed.h"

Serial bt (PA_9, PA_10);
char c; // declara c como char

void callback(void); // criando a função callback

int main() {
    bt.attach(&callback); // Toda vez que chega uma informação do módulo essa linha de código executa a função callback
    while(1) {
        // Código a ser executado
    }
          
}
void callback(void) 
{
    c = bt.getc();
}
```
## Buffer
As vezes é necessário enviar números como informação, para isso, utilizamos a função buffer para converter uma char para int. A ideia é que dentro do callback tenha uma condição que reconhece que a informação é um número e transorfma aquela informação para int. 
O código abaixo deseja enviar um número por bluetooth, para isso é necessário transformar o int em char. Também é importante, caso o código seja mais complexo e possua mais informações sendo mandadas, colocar a informação numérica entre dois caractéres para que o buffer entenda onde que ele precisa atuar, esse conceito ficará mais claro no código:
Esse código precisa de 3 botões e o intuito dele é fazer uma lógica de contagem que envia o valor final (numérico) por bleutooth. Caso não queira montar o sistema com 3 botões você pode apenas enviar diretamente um número por bluetooth. 
```C++
#include "mbed.h"

Serial bt (PA_9, PA_10);
DigitalIn v_mais (PC_6); // Botão para soma
DigitalIn v_menos (PC_5); // Botão para subtração
DigitalIn reset (PC_2); // Botão para enviar informação

char c; // declara c como char
int v = 0;

void callback(void); // criando a função callback

int main() {
    bt.attach(&callback); // Toda vez que chega uma informação do módulo essa linha de código executa a função callback
    while(1)
    {
        while(reset == 0){
            if (v_mais == 1){ // Se o botão v_mais for precionado a variavel v aumenta uma unidade
                v++;
                wait_ms(500);
            }
            if (v_menos == 1 and v!=0)
            {
                v--;
                wait_ms(500);
            }
        }
        bt.printf("d%i:",v); // Envia quantidade de volume em formato de char, entre os caracteres "d" e ":"
        wait(2);
    }          
}
void callback(void) 
{
    c = bt.getc();
}
```
O código abaixo recebe a informação do bluetooth e filtra para formar um número e por fim printa o número na serial do computador.
```C++
#include "mbed.h"

Serial bt (PA_9, PA_10);
Serial pc(USBTX, USBRX); // serial do computador

char c; // declara c como char
int v_bt = 0;

void callback(void); // criando a função callback

int main() {
    bt.attach(&callback); // Toda vez que chega uma informação do módulo essa linha de código executa a função callback
    while(1)
    {
      
    }          
}
void callback(void) 
{
    c = bt.getc();
    if(c=='d'){               // indentifica que se for d, é necessario fazer um buffer pois a informação é um número
        char buffer[64];      // consegue converter até 64 caractéres 
        int index = 0;        // número do índice
        while(c != ':'){      // funciona enquanto o ele não reconhecer ":" que é o final da informação numérica
            c = bt.getc();
            buffer[index] = c; // adiciona argumento no buffer com base no valor de c
            index++;           // vai percorrendo o número e adicionando à lista "buffer" 
        }
        volume_bt = atoi(buffer); // converte char para int
        pc.printf("%i", v_bt);       // printa na serial do computador o número  
    } 
}
```
# Conclusão
Com isso, você está preparado para utilizar o módulo HC-05 para comunicação entre duas placas NUCLEO F401RE. Caso tenha qualquer dúvida em relação ao tutorial, sinta-se à vontade para contatar o autor. Espero que este tutorial tenha lhe ajudado. 
# Autor
- Felipe Lisbona Fuchs
- email: felipefuchs123@gmail.com
- Data 18/09/2024

  ## Colaboradores
  - Silvio Szafir
  - Fabio Ferraz Junior
  - Kaique Dognani
  - Raphael Carvalho Souza Pires
  - Hugo Lara Campos 

  
# Referências 
O primeiro link trata de como configurar um módulo utilizando o FT232RL, e o segundo explica como configurar dois módulos para que eles possam interagir entre si.

[utilizando a FT232RL e o Termite para configurar o HC-05](https://www.arduinoecia.com.br/modulo-bluetooth-hc-05-conversor-ftdi/)

[Configurando módulos como mestre e servo](https://howtomechatronics.com/tutorials/arduino/how-to-configure-pair-two-hc-05-bluetooth-module-master-slave-commands/)
