# FP_ARM

Resumen de la especificación utilizada en los algoritmos de Suma-Resta IEEE754

En primera instancia, es necesario mencionar que el funcionamiento de ya sea el algoritmo de suma, o el de resta, es exactamente el mismo, con la única diferenciación de un par de instrucciones adicionales en el algoritmo de resta para realizar un simple cambio de signo en el segundo operando desde el inicio.
Teniendo claro esto, el algoritmo es prácticamente el mismo. Este se divide en varias partes fundamentales. En primer lugar, se requiere realizar el procesamiento de las cadenas de 32 bits, asumiendo que estas vienen en un formato correcto de IEEE754.
Mediante el uso de corrimientos tanto a la derecha como a la izquierda sobre dicha cadena, se extrae el signo, sesgo y la mantisa.
Ejemplo:
Supongamos que se debe procesar la siguiente cadena de bits.
00111000001001111100010110101100

-Haciendo un corrimiento de 31 espacios a la derecha obtenemos el signo del operando.

00111000001001111100010110101100 → 0

Bit de signo: 0

-Para la obtención del exponente, es necesario realizar 2 corrimientos.
El primero para omitir el bit de signo. 

00111000001001111100010110101100 → 0111000001001111100010110101100

El segundo para omitir los bits de la mantisa.

0111000001001111100010110101100 → 01110000

bits de sesgo: 01110000

-Para obtener la mantisa, se realiza un corrimiento de 24 bits a la derecha para omitir los bits para el signo y exponente.

00111000001001111100010110101100 →01001111100010110101100

Para un mejor manejo del error, se realiza un corrimiento de 9 bits a la izquierda para incorporar bits de guarda a la derecha de la cadena

01001111100010110101100000000000 → 01001111100010110101100000000000

Mantisa: 01001111100010110101100000000000

Una vez obtenidos estos elementos para cada operando, se debe de evaluar los exponentes de cada uno de los operandos, con el fin de alinear la mantisa que cuente con el exponente menor, con respecto a la otra. Todo esto antes de realizarse el proceso ya sea de suma o resta de las mantisas.
Ejemplo:

Supongamos los siguientes operandos.

1) 0|10000001|11010000000000000000000
2) 0|10000100|11010010000000000000000

En primera instancia, se debe verificar la operación con el fin de verificar si el resultado será 0. Para esto se verifica primero que los signos sean opuestos, y tanto la mantisa y el sesgo sean iguales. En este caso, se produce el resultado inmediatamente.

En otro caso, se debe considerar la diferencia entre los exponentes en este caso es de 112=310
Por lo tanto se debe de alinear el operando con menor exponente, haciendo 3 corrimientos sobre la mantisa de este.
1.11010000000000000000000000000000
0.00111010010000000000000000000000

Posterior al alineamiento, se procede a realizar la suma o resta de las mantisas.
Continuando con el mismo ejemplo,

  1.11010000000000000000000000000000
+0.00111010010000000000000000000000
10.00001000010000000000000000000000

El signo del resultado se determina según los tamaños y el orden de la combinación de exponente-mantisa de cada operando.
Una vez realizada la operación, es necesario realizar la selección del exponente temporal para el resultado. Para esto, simplemente se elije el sesgo mayor entre los dos operandos.
En nuestro ejemplo, este sería el siguiente

Exponente: 10000100

Realizando esto, ya se tiene un primer resultado. Aún así, este debe someterse a un proceso de normalización. Para esto se realizan los corrimientos necesarios según cada caso, para dejar la mantisa según el estándar, y se suma o resta la cantidad de movimientos al sesgo según la dirección del corrimiento.
Ejemplo:

Normalización de la mantisa: 10.00001000010000000000000000000000→1.00000100001000000000000000000000
Ajuste del exponente: 10000100→ 10000101

Una vez conociendo el resultado parcial, se debe evaluar si los números obtenidos se salen de los intervalos representables por el estándar IEEE754.
En caso de que el sesgo llegue a ser 11111111, se entiende que el resultado tiende a infinito. Si es puntualmente hacia +infinito o -infinito, es determinado por el bit de signo.
En este caso, se debe ajustar la salida para mostrar el resultado según lo solicita el estándar IEEE754.
Para esto se mantiene el signo, se mantiene el sesgo, y los bits de la mantisa se ponen en “0”, quedando de la siguiente forma

01111111100000000000000000000000



Otro posible resultado a con

Posteriormente se procede a eliminar los bits de guarda ubicados al final de la mantisa.

00000100001000000000000000000000→ 00000100001000000000000

Para finalizar, se unen los resultados obtenidos en el signo, el sesgo y la mantisa en una sola cadena de bits.

Resultados Obtenidos

Para la verificación del funcionamiento de los algoritmos de suma-resta IEEE754, se consideran la utilización de ciertos operandos con ciertas condiciones específicas.

1) En el primero de los casos, se considera la suma de números “muy grandes”.
El caso más crítico a considerar, es la suma de dos operandos sumamente grandes, que se salgan del intervalo permitido en el estándar.
Se realizó la siguiente prueba de validación para este caso

3.4028235E38
+3.4028235E38
INFINITO

  01111111011111111111111111111111
+01111111011111111111111111111110
  01111111100000000000000000000000

El software generó una salida que coincide con la esperada, por lo que se considera que el funcionamiento para esta prueba es relativamente aceptable.


2) Otra de las pruebas a considerar, es el caso en que un operando es “muy grande” y el otro “muy pequeño”.
Para validar este funcionamiento se utilizó la siguiente prueba.

150002493
+0.00004
150002493.00004

  0|10011010|00011110000110110110100
+0|01110000|01001111100010110101100 
  0|10011010|00011110000110110110100

Nuevamente se obtiene un resultado que coincide con lo esperado.

3) Para probar una operación que se acerque a 0, se considera la siguiente operación que hace una resta de dos números significativamente grandes, pero muy cercanos (cuya diferencia es cercana a “0”)

3.4028235E38
+-3.4028233E38
+0

  01111111011111111111111111111111
+11111111011111111111111111111110 
00000000000000000000000000000000

Este resultado es el esperado, ya que es la representación de un número que tiende a +0

4) Otra operación considerada en el proceso de pruebas, es la resta de un número “muy pequeño” ligeramente mayor, con un número “muy pequeño”, ligeramente menor.

  0.00005
+-0.00004
  0.00001

  00111000010100011011011100010111
+10111000001001111100010110101100
  0|01101110|01001111100010110101100

En este caso también se obtiene el resultado esperado.

5) En un caso similar, pero invirtiendo los signos en los operandos utilizados en el ejemplo anterior, se pretende evaluar los resultados con signo negativo.

   0.00004
+-0.00005
   -0.00001

  00111000001001111100010110101100  
+10111000010100011011011100010111
  1|01101110|01001111100010110101100

En este caso se obtienen resultados que coinciden con los esperados.











6) Para realizar una prueba con un caso similar, pero con números mucho más grandes, se procedió con el siguiente ejemplo.

      5300500
+-150002493
−144701993

  0|10010101|01000011100001000101000
+1|10011010|00011110000110110110100
  1|10011010|00010011111111110100011

En este ejemplo, también se obtiene el resultado esperado.


Con base en estas pruebas, se determina que el algoritmo implementado, tiene un funcionamiento relativamente aceptable.
