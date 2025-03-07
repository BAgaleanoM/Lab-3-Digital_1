# Lab-3-Digital_1
## Introducción
En este laboratorio, mediante una FPGA, se intentó modelar un voltímetro. Este a su vez consistía en una parte más analógica donde un transformador de 20:1 pasaba la tensión de 120V a 6V; después se rectificaba la señal, luego un diodo Zener para regular el voltaje y de ahí al conversor analógico digital ADC0808. Esto nos permitirá volver la señal digital y, al procesarla en el embebido, visualizar la tensión en voltios.

Por otra parte, es importante resaltar que el apartado digital se trabajó y comprendió; sin embargo, físicamente fue difícil su implementación debido a unas fallas, pero se abordará como se realizó el intento, lo que se buscaba finalmente y los impedimentos que se tuvieron.

## Desafío

Este laboratorio contiene, como se dijo anteriormente, un componente físico importante. Se debe partir sabiendo que a Colombia llega una tensión directa al tomacorriente de 110 V, pero por variaciones y protección se escogió un transformador 20:1; así, si tomábamos dicho voltaje como 120 V, pasaría a 6 V.

Una vez que se ha reducido este voltaje, se busca pasar por un puente de diodos o rectificador la señal. Quedando DC se conecta a un par de resistencias, para de este modo tener un divisor de voltaje y regularlo con ayuda del Zener, de ahí se conectaba al módulo ADC0808, para que por cada Data Bus salieran 1 bit, siendo en total 8, esto en un rango de 0 a 255, para que al lograr medir la tensión y pasando a código binario a BCD indique los voltios.

**Rectificación**

El voltaje, teóricamente como se dijo, se reducirá a 6 V RMS, quedan en unos 8,48Vp. Al pasar por el puente de diodos, se pasará por un condensador para suavizado de la señal y el voltaje presente que llegará a las resistencias es de 7,08.

**Divisor de Tensión**

Como se menciona, se plantea un divisor de tensión sencillo entre dos resistencias. Si se usa un trimmer de 100 K ohms en la resistencia dos, que debe ser la de caída, donde la tensión debe ser de 3,3 V, se debe situar en  46,610 K Ohms, para conectarse al conversor ADC, dejando este circuito.

![ACOPLE](./ACOPLE.png)

**ADC 0808**

Este lo vamos a energizar con 5 voltios, fijando así también nuestra referencia, para poder iniciar la conversión y finalizarla. Se pone en corto START con EOC (End Of Conversion), energizamos también OE y ALE, para habilitar la entrega de datos digitales y almacenar la dirección respectivamente. De los pines 23-25 salen los selectores A-B-C, del menos al más significativo. Este selector va a los 3.3, las salidas análogas irán a los 5 V y las digitales a la FPGA y los 3 módulos de 7 segmentos.

![ADC](./ADC.png)


## Código

Cabe aclarar que el resultado no fue el esperado y no se obtuvo lo que se deseaba, sin embargo, aqui esta lo trabajado.


**BIN a BCD**

```
module bin_to_bcd(
    input [7:0] bin,  
    output reg [11:0] bcd 
);
    integer i;
    reg [19:0] shift_reg; 
    
    always @(*) begin
        shift_reg = {12'b0, bin}; 
        
        for (i = 0; i < 8; i = i + 1) begin
            if (shift_reg[15:12] >= 5)
                shift_reg[15:12] = shift_reg[15:12] + 3;
            if (shift_reg[19:16] >= 5)
                shift_reg[19:16] = shift_reg[19:16] + 3;
            
            shift_reg = shift_reg << 1; 
        end
        
        bcd = shift_reg[19:8];
    end
endmodule
```




  **Divisor Frecuencia**


```

// filename: divFreq.v
module freqDiv #(
    parameter integer FREQ_IN = 1000,
    parameter integer FREQ_OUT = 100,
    parameter integer INIT = 0
) (
    // Inputs and output ports
    input CLK_IN,
    output reg CLK_OUT = 0
);

  localparam integer COUNT = (FREQ_IN / FREQ_OUT) / 2;
  localparam integer SIZE = $clog2(COUNT);
  localparam integer LIMIT = COUNT - 1;

  // Declaración de señales [reg, wire]
  reg [SIZE-1:0] count = INIT;

  // Descripción del comportamiento
  always @(posedge CLK_IN) begin
    if (count == LIMIT) begin
      count   <= 0;
      CLK_OUT <= ~CLK_OUT;
    end else begin
      count <= count + 1;
    end
  end
endmodule
```

**Medidor de Voltaje**


No es el top, porque aunque integramos todo lo que se intento, no llegó a hacer lo de la práctica y se colocaron los puntos fisicos correspondientes.

```
module medidor_voltaje (
    input wire clk,
    input wire reset,
    input wire [7:0] adc_input,
    output reg [11:0] bcd_output
);

    wire clk_div;
    
    freqDiv #(
        .FREQ_IN(1000),
        .FREQ_OUT(100),
        .INIT(0)
    ) divisor (
        .CLK_IN(clk),
        .CLK_OUT(clk_div)
    );

    reg [7:0] adc_reg;

    always @(posedge clk_div or posedge reset) begin
        if (reset) begin
            adc_reg <= 8'b00000000;
        end else begin
            adc_reg <= adc_input;
        end
    end

    wire [11:0] bcd_converted;
    
    bin_to_bcd converter (
        .bin(adc_reg),
        .bcd(bcd_converted)
    );

    always @(*) begin
        bcd_output = bcd_converted;
    end

endmodule

module bin_to_bcd(
    input [7:0] bin,  
    output reg [11:0] bcd 
);
    integer i;
    reg [19:0] shift_reg; 
    
    always @(*) begin
        shift_reg = {12'b0, bin}; 
        
        for (i = 0; i < 8; i = i + 1) begin
            if (shift_reg[15:12] >= 5)
                shift_reg[15:12] = shift_reg[15:12] + 3;
            if (shift_reg[19:16] >= 5)
                shift_reg[19:16] = shift_reg[19:16] + 3;
            
            shift_reg = shift_reg << 1; 
        end
        
        bcd = shift_reg[19:8];
    end
endmodule


```

Puertos Físicos

```

# Reloj principal de la FPGA (12 MHz)
set_io clk 38

# Botón de reset externo
set_io reset 34

# Entradas del ADC (8 bits)
set_io adc_input[0] 19
set_io adc_input[1] 20
set_io adc_input[2] 21
set_io adc_input[3] 22
set_io adc_input[4] 23
set_io adc_input[5] 24
set_io adc_input[6] 25
set_io adc_input[7] 26

# Salida BCD (12 bits)
set_io bcd_output[0] 9
set_io bcd_output[1] 10
set_io bcd_output[2] 11
set_io bcd_output[3] 12
set_io bcd_output[4] 15
set_io bcd_output[5] 16
set_io bcd_output[6] 17
set_io bcd_output[7] 18
set_io bcd_output[8] 28
set_io bcd_output[9] 29
set_io bcd_output[10] 31
set_io bcd_output[11] 32

```



## Dificultades

Como se ha dejado ver en este github, hubo dificultades notorias a la hora de hacer la práctica al completo, de esto cabe resaltar 2 cosas, primero que se intentó implementar un decodificador 74LS48, ya que este al conectar el módulo 7 segmentos conectando cada segmento (a-g) a su homónimo en el integrado y de ahí sale 4 pines, correspondientes a los 4 bits (ABCD), siendo de ese modo el A el menos significativo y el D él más. Todo esto, junto con el circuito descrito más arriba, incluso se instalaron en dos protoboards distintos, pero no acabó de funcionar.

Con lo comentado anteriormente, teniendo en cuenta de que se tenía todo separado y ordenado, fue un inesperado cuando nos dimos cuenta de que la FPGA dejó de funcionar y se había "quemado", hasta el momento no tenemos conclusiones muy claras de que pudo haber ocurrido, pero se apunta a un corto, mal funcionamiento del transformador o una alimentación errada, que incluso con las dos placas de prueba no logramos mitigar el error. Se adjuntan fotos de lo trabajado.


![ACOPLE](./ACOPLE.png)



