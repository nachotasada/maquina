# RTM32 - Tests de Instrucciones (STX4)

---

# Caso 1
## Descripción
Suma de dos registros con `ADD`. Verifico que el resultado se deposita correctamente en el destino y que los operandos no se modifican.

## Instructions
```
ADD $t0, $t1, $t2
```

## Precondiciones
- Setear `$t1` ($10) = `0x00000005`  (5)
- Setear `$t2` ($11) = `0x00000003`  (3)
- `$t0` ($12) puede tener cualquier valor previo (va a ser sobreescrito)

## Code
```asm
ADD $t0, $t1, $t2     ; $t0 = $t1 + $t2
```

## Postcondiciones
- Ver en el panel de registros que `$t0` ($12) = `0x00000008` (8)
- `$t1` sigue siendo `0x00000005`
- `$t2` sigue siendo `0x00000003`

## Conclusiones
Anduvo. El resultado 5 + 3 = 8 apareció correctamente en `$t0`. Los registros fuente no fueron modificados, lo cual es el comportamiento esperado para una instrucción tipo R.

---

# Caso 2
## Descripción
Resta con `SUB`. Testeo también el caso donde el resultado es negativo para verificar que trabaja con complemento a dos.

## Instructions
```
SUB $t0, $t1, $t2
```

## Precondiciones
- Setear `$t1` ($10) = `0x00000003`  (3)
- Setear `$t2` ($11) = `0x00000007`  (7)

## Code
```asm
SUB $t0, $t1, $t2     ; $t0 = $t1 - $t2 = 3 - 7 = -4
```

## Postcondiciones
- `$t0` debe mostrar `0xFFFFFFFC` que es -4 en complemento a dos de 32 bits

## Conclusiones
Anduvo. El resultado -4 se almacenó como `0xFFFFFFFC`, confirmando que la ALU opera correctamente en complemento a dos.

---

# Caso 3
## Descripción
Suma con inmediato `ADDI`. Útil para inicializar registros o incrementar contadores. Testeo con valor positivo y con valor negativo (ya que imm está en complemento a dos de 17 bits).

## Instructions
```
ADDI $t0, $zero, 42
ADDI $t1, $t0, -10
```

## Precondiciones
- No se necesita setear nada previamente; `$zero` siempre es 0.

## Code
```asm
ADDI $t0, $zero, 42   ; $t0 = 0 + 42 = 42
ADDI $t1, $t0, -10    ; $t1 = 42 + (-10) = 32
```

## Postcondiciones
- `$t0` = `0x0000002A` (42)
- `$t1` = `0x00000020` (32)

## Conclusiones
⚠️ **Bug del simulador — instrucción no testeable.** Al ejecutar ADDI con encoding `0x0814002A`, el simulador generó una excepción interna (CAUSE = 3) en lugar de ejecutar la operación. El simulador entró en estado inválido y fue necesario reiniciarlo. Este comportamiento es consistente con el bug de ANDI documentado por el profesor. La instrucción ADDI se omite del testeo funcional por este motivo.

---

# Caso 4
## Descripción
`AND` lógico bit a bit entre dos registros. Sirve para aplicar máscaras. Verifico que sólo los bits que están en 1 en ambos operandos quedan en 1.

## Instructions
```
AND $t0, $t1, $t2
```

## Precondiciones
- Setear `$t1` = `0x0000FF0F`
- Setear `$t2` = `0x00FF00FF`

## Code
```asm
AND $t0, $t1, $t2     ; $t0 = 0x0000FF0F & 0x00FF00FF
```

## Postcondiciones
- `$t0` = `0x0000000F`

## Conclusiones
Anduvo. El AND bit a bit produjo `0x0000000F`, que es exactamente la intersección de los bits en 1 de ambos valores.

---

# Caso 5
## Descripción
`OR` lógico bit a bit. Testeo que la unión de bits funciona correctamente, en particular que OR con `$zero` actúa como un MOV.

## Instructions
```
OR $t0, $t1, $t2
OR $s0, $t1, $zero
```

## Precondiciones
- Setear `$t1` = `0xA0A0A0A0`
- Setear `$t2` = `0x0F0F0F0F`

## Code
```asm
OR $t0, $t1, $t2      ; $t0 = 0xA0A0A0A0 | 0x0F0F0F0F
OR $s0, $t1, $zero    ; $s0 = $t1 (equivalente a MOV)
```

## Postcondiciones
- `$t0` = `0xAFAFAFAF`
- `$s0` = `0xA0A0A0A0`

## Conclusiones
Anduvo. El OR combinó correctamente los bits. El segundo OR con `$zero` funcionó como un MOV, copiando `$t1` a `$s0` sin alterar su valor.

---

# Caso 6
## Descripción
`XOR` lógico bit a bit. Verifico la propiedad de que `X XOR X = 0`, usada como truco para poner en cero un registro sin usar una constante.

## Instructions
```
XOR $t0, $t1, $t2
XOR $s0, $t3, $t3
```

## Precondiciones
- Setear `$t1` = `0xDEADBEEF`
- Setear `$t2` = `0xFFFFFFFF`
- `$t3` puede tener cualquier valor

## Code
```asm
XOR $t0, $t1, $t2     ; $t0 = 0xDEADBEEF ^ 0xFFFFFFFF  (inversión de bits)
XOR $s0, $t3, $t3     ; $s0 = $t3 ^ $t3 = 0
```

## Postcondiciones
- `$t0` = `0x21524110` (todos los bits invertidos)
- `$s0` = `0x00000000`

## Conclusiones
Anduvo. El XOR con `0xFFFFFFFF` actuó como NOT, y el XOR consigo mismo dio 0 como se esperaba.

---

# Caso 7
## Descripción
`NOR` lógico. `NOR(A, B) = NOT(A OR B)`. Testeo que con `$zero` como segundo operando actúa como un NOT.

## Instructions
```
NOR $t0, $t1, $zero
```

## Precondiciones
- Setear `$t1` = `0x0000FFFF`

## Code
```asm
NOR $t0, $t1, $zero   ; $t0 = ~($t1 | 0) = ~$t1
```

## Postcondiciones
- `$t0` = `0xFFFF0000`

## Conclusiones
Anduvo. NOR con `$zero` invirtió todos los bits de `$t1`, funcionando como NOT. Esta es la forma estándar de implementar NOT en arquitecturas que no tienen instrucción dedicada.

---

# Caso 8
## Descripción
Desplazamiento lógico izquierdo `SLL` (Shift Left Logical). Cada posición de desplazamiento equivale a multiplicar por 2. Testeo con desplazamiento de 3 bits (×8).

## Instructions
```
SLL $t0, $zero, $t1, 3
```
*(En sintaxis del ensamblador: `SLL $t0, $t1, 3` — rd=destino, rt=fuente, aux=cantidad)*

## Precondiciones
- Setear `$t1` = `0x00000001`

## Code
```asm
SLL $t0, $t1, 3       ; $t0 = $t1 << 3 = 1 << 3 = 8
```

## Postcondiciones
- `$t0` = `0x00000008`

## Conclusiones
Anduvo. El desplazamiento de 3 bits a la izquierda multiplicó por 8. Los bits que "salieron" por la izquierda se descartan y los que entran por la derecha son ceros.

---

# Caso 9
## Descripción
Desplazamiento lógico derecho `SRL` (Shift Right Logical). Los bits que entran por la izquierda son ceros (sin extensión de signo). Verifico con un valor negativo.

## Instructions
```
SRL $t0, $t1, 1
```

## Precondiciones
- Setear `$t1` = `0x80000000` (bit de signo en 1, valor negativo en signed)

## Code
```asm
SRL $t0, $t1, 1       ; $t0 = $t1 >> 1 (lógico, entra 0)
```

## Postcondiciones
- `$t0` = `0x40000000` (el bit de signo se convirtió en 0, entró un 0 por la izquierda)

## Conclusiones
Anduvo. El SRL desplazó el bit de signo a la derecha y colocó un 0 en el MSB, confirmando que es un desplazamiento lógico (sin considerar el signo).

---

# Caso 10
## Descripción
Desplazamiento aritmético derecho `SRA` (Shift Right Arithmetic). A diferencia del SRL, preserva el signo (extiende el bit más significativo). Testeo con un número negativo.

## Instructions
```
SRA $t0, $t1, 2
```

## Precondiciones
- Setear `r12` = `0x80000000` (-2147483648) — este es el campo **rt** del encoding, que el hardware usa como fuente real

> **Discrepancia con el manual:** el manual indica `R[rd] = R[rs]` como fuente, pero el hardware usa `R[rt]`. El valor debe estar en rt (r12), no en rs.

## Code
Machine code: `0x0018A102`
```
[31:27]=00000 R-type | [26:22]=00000 rs=r0 | [21:17]=01100 rt=r12 | [16:12]=01010 rd=r10 | [11:7]=00010 aux=2 | [5:0]=000010 func=SRA
```

## Postcondiciones
- `r10` = `0xE0000000` (los dos bits de mayor peso son 1, signo propagado)

## Conclusiones
Anduvo. El SRA propagó el bit de signo hacia la derecha, a diferencia del SRL que habría colocado ceros. Esto equivale a dividir por 4 preservando el signo. **Discrepancia detectada entre el manual y el hardware:** el manual especifica `R[rs]` como registro fuente, pero el hardware efectivamente lee `R[rt]`. Al colocar el valor en rs el resultado fue 0; al colocarlo en rt (r12) el resultado fue correcto (`0xE0000000`).

---

# Caso 11
## Descripción
`SLT` (Set Less Than). Pone 1 en el destino si rs < rt (con signo), 0 en caso contrario. Testeo los tres casos: menor, igual y mayor.

## Instructions
```
SLT $t0, $t1, $t2
SLT $t3, $t2, $t2
SLT $t4, $t2, $t1
```

## Precondiciones
- Setear `$t1` = `0x00000003`  (3)
- Setear `$t2` = `0x00000007`  (7)

## Code
```asm
SLT $t0, $t1, $t2     ; 3 < 7  → $t0 = 1
SLT $t3, $t2, $t2     ; 7 < 7  → $t3 = 0 (iguales)
SLT $t4, $t2, $t1     ; 7 < 3  → $t4 = 0
```

## Postcondiciones
- `$t0` = `0x00000001`
- `$t3` = `0x00000000`
- `$t4` = `0x00000000`

## Conclusiones
Anduvo. Los tres casos cubrieron menor, igual y mayor. El resultado fue 1 solo cuando la condición es estrictamente menor.

---

# Caso 12
## Descripción
Guardado y carga de una palabra entera `SW` / `LW`. Escribo un valor en memoria y luego lo vuelvo a leer para verificar que se almacenó correctamente.

## Instructions
```
SW $t0, $t1, 0
LW $t2, $t1, 0
```

## Precondiciones
- Setear `$t0` = `0xCAFE1234` (valor a guardar)
- Setear `$t1` = dirección de memoria alineada disponible en el simulador (por ejemplo `0x00001000`)
- Verificar que `0x00001000` es múltiplo de 4 (requisito de alineación de LW/SW)

## Code
```asm
SW $t0, $t1, 0        ; M[$t1 + 0] = $t0  →  guarda 0xCAFE1234 en 0x1000
LW $t2, $t1, 0        ; $t2 = M[$t1 + 0]  →  carga desde 0x1000
```

## Postcondiciones
- En el visor de memoria, la posición `0x00001000` debe mostrar `0xCAFE1234`
- `$t2` debe ser `0xCAFE1234`

## Conclusiones
Anduvo. El dato se escribió en memoria y se recuperó sin alteraciones. El offset 0 funcionó correctamente.

---

# Caso 13
## Descripción
`SW` / `LW` con offset no cero. Verifico que la dirección efectiva EA = R[rs] + SE(imm) se calcula correctamente.

## Instructions
```
SW $t0, $t1, 4
LW $t2, $t1, 4
```

## Precondiciones
- Setear `$t0` = `0xABCD0000`
- Setear `$t1` = `0x00001000` (base)

## Code
```asm
SW $t0, $t1, 4        ; M[0x1000 + 4] = M[0x1004] = 0xABCD0000
LW $t2, $t1, 4        ; $t2 = M[0x1004]
```

## Postcondiciones
- Posición `0x00001004` en memoria = `0xABCD0000`
- `$t2` = `0xABCD0000`
- Posición `0x00001000` no fue modificada (del caso anterior debería seguir siendo `0xCAFE1234`)

## Conclusiones
Anduvo. El offset 4 se sumó correctamente a la base, apuntando a la palabra siguiente sin pisar el dato anterior.

---

# Caso 14
## Descripción
`SB` / `LB` — guardar y cargar un byte con extensión de signo. Verifico que solo el byte menos significativo se escribe, y que al leer se hace extensión de signo.

## Instructions
```
SB $t0, $t1, 0
LB $t2, $t1, 0
```

## Precondiciones
- Setear `$t0` = `0x000000FF`  (byte = 0xFF = -1 signed)
- Setear `$t1` = `0x00001008`

## Code
```asm
SB $t0, $t1, 0        ; M[0x1008][7:0] = 0xFF
LB $t2, $t1, 0        ; $t2 = sign_extend(0xFF) = 0xFFFFFFFF (-1)
```

## Postcondiciones
- Byte en `0x1008` = `0xFF`
- `$t2` = `0xFFFFFFFF` (extensión de signo del byte 0xFF)

## Conclusiones
Anduvo. `SB` escribió solo el byte bajo. `LB` leyó el byte y lo extendió con signo, convirtiendo `0xFF` en `0xFFFFFFFF`.

---

# Caso 15
## Descripción
`LBU` — cargar byte sin extensión de signo. Misma situación que el caso anterior pero sin extender el signo; los bits superiores deben quedar en cero.

## Instructions
```
LBU $t2, $t1, 0
```

## Precondiciones
- Mismas del Caso 14: byte `0xFF` ya guardado en `0x1008`
- `$t1` = `0x00001008`

## Code
```asm
LBU $t2, $t1, 0       ; $t2 = {24'b0, M[0x1008][7:0]} = 0x000000FF
```

## Postcondiciones
- `$t2` = `0x000000FF` (255, sin extensión de signo)

## Conclusiones
Anduvo. A diferencia de `LB`, el LBU rellenó los bits superiores con cero en lugar de propagar el bit de signo. El mismo byte `0xFF` resultó en 255 en lugar de -1.

---

# Caso 16
## Descripción
`BEQ` — salto condicional si dos registros son iguales. Verifico que el salto ocurre cuando son iguales y que NO ocurre cuando son distintos.

## Instructions
```
BEQ $t0, $t1, 2
ADD $t2, $zero, $zero
ADD $t3, $zero, $zero
```

## Precondiciones
- Setear `$t0` = `0x00000010`
- Setear `$t1` = `0x00000010` (igual a $t0 para que el salto ocurra)
- PC apuntando a la instrucción BEQ

## Code
```asm
BEQ $t0, $t1, 2       ; si $t0 == $t1, salta 2 instrucciones hacia adelante
ADD $t2, $zero, $zero  ; instrucción que se SALTEA
ADD $t3, $zero, $zero  ; instrucción que se SALTEA también
; <--- PC debería llegar acá después del salto
```

## Postcondiciones
- Si el salto ocurrió: PC avanzó saltando las dos instrucciones `ADD`. `$t2` y `$t3` no fueron modificados.
- Verificar en el debug que el PC efectivamente saltó: PC_nuevo = PC_BEQ + 4 + 4×2 = PC_BEQ + 12

## Conclusiones
Anduvo. Como `$t0 == $t1`, el BEQ ejecutó el salto y las dos instrucciones ADD intermedias no se ejecutaron. Al cambiar `$t1` a un valor diferente y repetir, el BEQ no saltó y el flujo siguió normalmente.

---

# Caso 17
## Descripción
`BNE` — salto si los registros son distintos. Lo testeo con valores claramente diferentes y también verifico que no salta cuando son iguales.

## Instructions
```
BNE $t0, $t1, 1
SUB $t2, $t1, $t0
```

## Precondiciones
- Setear `$t0` = `0x00000001`
- Setear `$t1` = `0x00000002`

## Code
```asm
BNE $t0, $t1, 1       ; $t0 != $t1 → salta 1 instrucción
SUB $t2, $t1, $t0     ; esta instrucción se saltea
; continua acá
```

## Postcondiciones
- El BNE saltó: el SUB no se ejecutó, `$t2` conserva su valor previo
- PC avanzó PC_BNE + 4 + 4×1 = PC_BNE + 8

## Conclusiones
Anduvo. El salto se realizó porque los valores eran distintos. Al setear `$t0 = $t1` y repetir, el BNE no saltó y el SUB sí se ejecutó.

---

# Caso 18
## Descripción
`BLT` — salto si rs < rt (signed). Testeo con un valor negativo en rs para verificar que la comparación es con signo.

## Instructions
```
BLT $t0, $t1, 2
ADD $t2, $zero, $zero
```

## Precondiciones
- Setear `$t0` = `0xFFFFFFFF` (-1 en complemento a dos)
- Setear `$t1` = `0x00000001`  (1)

## Code
```asm
BLT $t0, $t1, 2       ; -1 < 1 → debe saltar
ADD $t2, $zero, $zero  ; se saltea
```

## Postcondiciones
- El salto ocurrió, `$t2` no fue modificado

## Conclusiones
Anduvo. La comparación trató correctamente a `0xFFFFFFFF` como -1 (signed), menor que 1. Si la comparación fuera sin signo, `0xFFFFFFFF` sería mayor, y el salto no ocurriría.

---

# Caso 19
## Descripción
`MUL` — multiplicación con signo, resultado de 32 bits bajos. Testeo caso básico y luego un caso donde el resultado excede 32 bits para ver que solo guarda los bits bajos.

## Instructions
```
MUL $t0, $t1, $t2
```

## Precondiciones
- Setear `$t1` = `0x00000006`  (6)
- Setear `$t2` = `0x00000007`  (7)

## Code
```asm
MUL $t0, $t1, $t2     ; $t0 = (6 × 7)[31:0] = 42
```

## Postcondiciones
- `$t0` = `0x0000002A` (42)

## Conclusiones
Anduvo. 6 × 7 = 42 = `0x2A`. El resultado entró perfectamente en 32 bits.

---

# Caso 20
## Descripción
`MULH` — parte alta de la multiplicación con signo (bits 63:32). Verifico que captura el overflow de 32 bits correctamente.

## Instructions
```
MULH $t0, $t1, $t2
```

## Precondiciones
- Setear `$t1` = `0x00010000`  (65536)
- Setear `$t2` = `0x00010000`  (65536)
- El producto = 65536² = 4294967296 = `0x100000000`, que excede 32 bits

## Code
```asm
MULH $t0, $t1, $t2    ; $t0 = (65536 × 65536)[63:32] = 1
```

## Postcondiciones
- `$t0` = `0x00000001` (la parte alta del resultado de 64 bits)

## Conclusiones
Anduvo. `MULH` capturó el bit 32 del producto, que es la parte que `MUL` hubiera descartado. Combinando `MUL` y `MULH` se puede obtener el producto completo de 64 bits.

---

# Caso 21
## Descripción
`DIV` — división entera con signo. Testeo el cociente y también el caso de división negativa.

## Instructions
```
DIV $t0, $t1, $t2
```

## Precondiciones
- Setear `$t1` = `0x00000014`  (20)
- Setear `$t2` = `0x00000004`  (4)

## Code
```asm
DIV $t0, $t1, $t2     ; $t0 = 20 / 4 = 5
```

## Postcondiciones
- `$t0` = `0x00000005`

## Conclusiones
Anduvo. 20 ÷ 4 = 5, resultado exacto sin resto.

---

# Caso 22
## Descripción
`REST` — resto de la división entera (módulo). Complemento al Caso 21.

## Instructions
```
REST $t0, $t1, $t2
```

## Precondiciones
- Setear `$t1` = `0x00000011`  (17)
- Setear `$t2` = `0x00000005`  (5)

## Code
```asm
REST $t0, $t1, $t2    ; $t0 = 17 % 5 = 2
```

## Postcondiciones
- `$t0` = `0x00000002`

## Conclusiones
Anduvo. 17 mod 5 = 2 (ya que 5×3 = 15 y 17-15 = 2).

---

# Caso 23
## Descripción
`LUI` — carga inmediato en los 16 bits altos. Indispensable para construir constantes de 32 bits. Testeo que los 16 bits bajos quedan en cero.

## Instructions
```
LUI $t0, 0xABCD
```

## Precondiciones
- Ninguna (solo necesita el registro destino)

## Code
```asm
LUI $t0, 0xABCD       ; $t0 = 0xABCD0000
```

## Postcondiciones
- `$t0` = `0xABCD0000`

## Conclusiones
Anduvo. El inmediato `0xABCD` se cargó en los 16 bits más significativos y los 16 bajos quedaron en 0.

---

# Caso 24
## Descripción
`LUI` + `ORI` — técnica estándar para cargar una constante de 32 bits arbitraria en un registro.

## Instructions
```
LUI $t0, 0xDEAD
ORI $t0, $t0, 0xBEEF
```

## Precondiciones
- Ninguna

## Code
```asm
LUI $t0, 0xDEAD       ; $t0 = 0xDEAD0000
ORI $t0, $t0, 0xBEEF  ; $t0 = 0xDEAD0000 | 0x0000BEEF = 0xDEADBEEF
```

## Postcondiciones
- Después del LUI: `$t0` = `0xDEAD0000`
- Después del ORI: `$t0` = `0xDEADBEEF`

## Conclusiones
Anduvo. Esta es la forma canónica de cargar cualquier constante de 32 bits. El LUI establece la mitad alta y el ORI completa la mitad baja sin tocar los bits ya puestos.

---

# Caso 25
## Descripción
`SLTI` — set less than immediate (con signo). Verifico la comparación con una constante inmediata negativa.

## Instructions
```
SLTI $t0, $t1, -1
SLTI $t2, $t1, 100
```

## Precondiciones
- Setear `$t1` = `0x00000000` (0)

## Code
```asm
SLTI $t0, $t1, -1     ; 0 < -1 ?  → NO  → $t0 = 0
SLTI $t2, $t1, 100    ; 0 < 100 ? → SÍ  → $t2 = 1
```

## Postcondiciones
- `$t0` = `0x00000000`
- `$t2` = `0x00000001`

## Conclusiones
Anduvo. El inmediato -1 se extendió con signo correctamente. 0 no es menor que -1 (signed), y sí es menor que 100.

---

# Caso 26
## Descripción
`J` — salto incondicional a dirección absoluta. Verifico que el PC cambia al destino correcto y que la instrucción siguiente al J no se ejecuta.

## Instructions
```
J destino
ADD $t0, $t1, $t2    ; no debe ejecutarse
destino:
ADD $s0, $zero, $zero
```

## Precondiciones
- Programa cargado con las instrucciones en orden consecutivo
- `$t0`, `$s0` con valores distintos de cero para ver si cambian

## Code
```asm
J destino             ; salto incondicional
ADD $t0, $t1, $t2    ; instrucción saltada (no ejecutada)
destino:
ADD $s0, $zero, $zero ; PC llega aquí
```

## Postcondiciones
- `$t0` no fue modificado (el ADD después del J no se ejecutó)
- `$s0` = 0 (el ADD en destino sí se ejecutó)
- PC avanzó al label `destino`

## Conclusiones
Anduvo. El J transfirió el control incondicionalmente. La instrucción siguiente fue saltada. La dirección del salto se calculó como `{PC+4[31:29], address, 2'b0}`.

---

# Caso 27
## Descripción
`SH` / `LH` — guardar y cargar media palabra (16 bits) con extensión de signo.

## Instructions
```
SH $t0, $t1, 0
LH $t2, $t1, 0
```

## Precondiciones
- Setear `$t0` = `0x0000FFFF`  (half word = 0xFFFF = -1 signed 16-bit)
- Setear `$t1` = `0x00001010`  (dirección alineada a 2 bytes)

## Code
```asm
SH $t0, $t1, 0        ; M[0x1010][15:0] = 0xFFFF
LH $t2, $t1, 0        ; $t2 = sign_extend(0xFFFF) = 0xFFFFFFFF
```

## Postcondiciones
- Los 2 bytes en `0x1010` = `0xFFFF`
- `$t2` = `0xFFFFFFFF`

## Conclusiones
Anduvo. `SH` escribió solo los 16 bits bajos. `LH` los cargó y extendió el signo: `0xFFFF` con el bit 15 en 1 se convirtió en `0xFFFFFFFF`.

---

# Caso 28
## Descripción
`SLLR` — shift left lógico con cantidad variable en registro (a diferencia de SLL donde la cantidad es inmediata). Verifico que solo se usan los 5 bits bajos del registro de cantidad.

## Instructions
```
SLLR $t0, $t1, $t2
```

## Precondiciones
- Setear `$t1` = `0x00000001`
- Setear `$t2` = `0x00000004`  (desplazamiento de 4 bits, solo se usan `$t2[4:0]`)

## Code
```asm
SLLR $t0, $t1, $t2    ; $t0 = $t1 << $t2[4:0] = 1 << 4 = 16
```

## Postcondiciones
- `$t0` = `0x00000010` (16)

## Conclusiones
Anduvo. El desplazamiento dinámico desde registro funciona igual que el inmediato pero permite variar la cantidad en tiempo de ejecución.

---

> **Nota:** Las instrucciones TRAP, RFT, CFS y CTS no se testean en este documento ya que requieren configuración del sistema de excepciones e interrupciones, lo cual agrega complejidad innecesaria para la validación básica del ISA. Las instrucciones ANDI y ADDI presentan bugs en el simulador: ambas generan una excepción interna (CAUSE = 3) al ejecutarse, dejando el simulador en estado inválido. ANDI está documentada por el profesor como bug conocido; ADDI exhibe el mismo comportamiento.
