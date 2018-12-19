# Funções e pilha

Apesar de não estudarmos todos os aspectos da linguagem Assembly, alguns assuntos são de extrema importância, mesmo para os fundamentos da engenharia reversa de software. Um deles é como funcionam as funções criadas em um programa e suas chamadas, que discutiremos agora.

## O que é uma função?

Basicamente, uma função é um **bloco de código reutilizável** num programa. Tal bloco faz-se útil quando um determinado conjunto de instruções precisa ser invocado em vários pontos do programa. Por exemplo, suponha um programa que precise converter a temperatura de Fahrenheit para Celsius várias vezes no decorrer de seu código:

{% code-tabs %}
{% code-tabs-item title="fahrenheit2celsius.py" %}
```python
fahrenheit = 230.4
celsius = (fahrenheit - 32) * 5 / 9
print(celsius)

fahrenheit = 130.3
celsius = (fahrenheit - 32) * 5 / 9
print(celsius)

fahrenheit = 90.1
celsius = (fahrenheit - 32) * 5 / 9
print(celsius)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

O programa acima funciona e a saída é, conforme esperada:

```text
110.22222222222223
54.611111111111114
32.27777777777778
```

No entanto, é pouco prático, pois repetimos o mesmo código várias vezes. Além disso, uma versão compilada fica maior em _bytes_. Também prejudica a manutenção do código pois se o programador precisar fazer uma alteração no cálculo, vai ter que alterar em todos eles. É aí que entram as funções. Veja:

{% code-tabs %}
{% code-tabs-item title="fahrenheit2celsius.py" %}
```python
def fahrenheit2celsius(fahrenheit):
    return (fahrenheit - 32) * 5 / 9

celsius = fahrenheit2celsius(230.4)
print(celsius)

celsius = fahrenheit2celsius(130.3)
print(celsius)

celsius = fahrenheit2celsius(90.1)
print(celsius)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

A saída é a mesma, mas agora o programa está utilizando uma função, onde o cálculo só foi definido uma única vez e toda vez que for necessário, o programa a chama.

Uma função tem:

1. Argumentos, também chamados de parâmetros, que são os dados que a função recebe, necessários para cumprir seu propósito.
2. Retorno, que é o resultado da conclusão do seu propósito, seja bem sucedida ou não.
3. Um nome \(na visão do programador\) ou um endereço de memória \(na visão do processador\).

Agora cabe a nós estudar como isso tudo funciona em baixo nível. Pronto? 🤷‍♂️

{% hint style="info" %}
Nos primórdios da computação as funções eram chamadas de **procedimentos** \(_procedures_\). Em algumas linguagens de programação, no entanto, possuem tanto funções quanto procedimentos. Estes últimos são "funções que não retornam nada". Já no paradigma da programação orientada a objetos \(POO\), as funções de uma classe são chamadas de **métodos**.
{% endhint %}

## Funções em Assembly

Em baixo nível, uma função é implementada basicamente num bloco que não será executado até ser chamado por uma instrução CALL. Ao final de uma instrução, encontramos normalmente a instrução RET. Vamos analisar uma função simples de soma para entender:

```c
#include <stdio.h>

int soma(int x, int y) {
  return x+y;
}

int main(void) {
  printf("%d\n", soma(3,4));
  return 0;
}
```

Olha como ela fica compilada no Linux em 32-bits:

```text
0804840b <soma>:
 804840b:       55                      push   ebp
 804840c:       89 e5                   mov    ebp,esp
 804840e:       8b 55 08                mov    edx,DWORD PTR [ebp+0x8]
 8048411:       8b 45 0c                mov    eax,DWORD PTR [ebp+0xc]
 8048414:       01 d0                   add    eax,edx
 8048416:       5d                      pop    ebp
 8048417:       c3                      ret

08048418 <main>:
...
 8048429:       6a 04                   push   0x4
 804842b:       6a 03                   push   0x3
 804842d:       e8 d9 ff ff ff          call   804840b <soma>
 8048432:       83 c4 08                add    esp,0x8
```

Removi partes do código intencionalmente, pois o objetivo neste momento é apresentar as instruções que implementam as chamadas de função. Por hora, você só precisa entender que a instrução CALL \(no endereço 0x804842d em nosso exemplo\) chama as funções e a instrução RET \(em 0x8048417\) retorna para a instrução imediatamente após a CALL \(0x8048432\), para que a execução continue.

### A pilha de memória

A memória RAM para um processo é dividida em áreas com diferentes propósitos. Uma delas é a pilha, ou _stack_.

Essa área de memória funciona de forma com que o que é colocado lá fique no topo e o último dado colocado na pilha seja o primeiro a ser retirado, como uma pilha de pratos ou de cartas de baralho. Esse método é conhecido por LIFO \(_Last In First Out_\).

Seu principal uso é no uso de funções, tanto para passagem de argumentos \(parâmetros da função\) quanto para alocação de variáveis locais à função \(que só existem enquanto a função executa\).

Na IA-32, a pilha é alinhada em 4 _bytes_ \(32-bits\). Por consequência todos os seus endereços também o são. Logo, se novos dados são colocados na pilha \(empilhados\), o endereço do topo é **decrementado** em 4 unidades. Se um dado for desempilhado, o endereço é **incrementado** em 4 unidades. Perceba a lógica invertida, porque a pilha começa num endereço alto e vai descendo.

Existem dois registradores intimamente associados com a pilha de memória alocada para um processo. São eles:

* O ESP, que aponta para o topo da pilha.
* O EBP, que aponta para a base do _stack frame_.

A instrução CALL faz duas coisas:

1. Coloca o endereço da próxima instrução na pilha de memória \(no caso do exemplo, 0x8048432\).
2. Coloca o seu parâmetro, ou seja, o endereço da função a ser chamada, no registrador EIP \(no exemplo é o endereço 0x804840b\).

Por conta dessa atualização do EIP, o fluxo é desviado para o endereço da função chamada. A ideia de colocar o endereço da próxima instrução na pilha é para o processador saber para onde tem que voltar quando a função terminar. E falando em terminar, a estrela do fim da festa é a instrução RET \(de _RETURN_\). Ela faz uma única coisa:

1. Retira um valor do topo da pilha e coloca no EIP.

{% hint style="danger" %}
AINDA ESTAMOS TRABALHANDO NESTA SEÇÃO
{% endhint %}



* O principal uso da pilha \(stack\) é para as funções armazenarem variáveis locais e também para passagem de parâmetros \(dependendo da calling convention\).
* Alinhas em 4 bytes \(32-bits\)
  * Cresce para baixo
  * Prólogo e epílogo de funções
* Recuperação de parâmetros
* Instruções de manipulação da pilha:
  * PUSH copia um valor para a pilha \(empilha\) e decremente o registrador ESP em 4. Ex.: PUSH 0x51
  * POP recupera um valor da pilha \(desempilha\) para algum lugar e incrementa ESP em 4. Exemplo: POP EBX.
  * CALL empilha o endereço da próxima instrução e muda o EIP para o destino dela.
  * RET desempilha o endereço no topo da pilha para EIP

