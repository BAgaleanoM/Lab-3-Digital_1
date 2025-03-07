# Lab-3-Digital_1
## Introducción
En este laboratorio, mediante una FPGA, se intentó modelar un voltímetro. Este a su vez consistía en una parte más analógica donde un transformador de 20:1 pasaba la tensión de 120V a 6V; después se rectificaba la señal, luego un diodo Zener para regular el voltaje y de ahí al conversor analógico digital ADC0808. Esto nos permitirá volver la señal digital y, al procesarla en el embebido, visualizar la tensión en voltios.

Por otra parte, es importante resaltar que el apartado digital se trabajó y comprendió; sin embargo, físicamente fue difícil su implementación debido a unas fallas, pero se abordará como se realizó el intento, lo que se buscaba finalmente y los impedimentos que se tuvieron.

## Desafío

Este laboratorio contiene, como se dijo anteriormente, un componente físico importante. Se debe partir sabiendo que a Colombia llega una tensión directa a la pared de 110 V, pero por variaciones y protección se escogió un transformador 20:1; así, si tomábamos dicho voltaje como 120 V, pasaría a 6 V.

Una vez que se ha reducido este voltaje, se busca pasar por un puente de diodos o rectificador la señal. Quedando DC se conecta a un par de resistencias, para de este modo tener un divisor de voltaje y regularlo con ayuda del Zener, de ahí se conectaba al módulo ADC0808, para que por cada Data Bus salieran 1 bit, siendo en total 8, esto en un rango de 0 a 255, para que al lograr medir la tensión y pasando a código binario a BCD indique los voltios.

*
