# RTM32 - Tests de Instrucciones (STX4)

---

# Caso 1
## Descripción
`ADD` — suma de dos registros.

## Precondiciones
- `$t1` = `0x00000005`
- `$t2` = `0x00000003`

## Code
```asm
ADD $t0, $t1, $t2
```

## Postcondiciones
- `$t0` = `0x00000008`

## Conclusiones
Anduvo. 5 + 3 = 8 en `$t0`.

---

# Caso 2
## Descripción
`SUB` — resta. Resultado negativo en complemento a dos.

## Precondiciones
- `$t1` = `0x00000003`
- `$t2` = `0x00000007`

## Code
```asm
SUB $t0, $t1, $t2
```

## Postcondiciones
- `$t0` = `0xFFFFFFFC`

## Conclusiones
Anduvo. 3 - 7 = -4 representado como `0xFFFFFFFC`.

---

# Caso 3
## Descripción
`ADDI` — suma con inmediato de 17 bits con signo.

## Conclusiones
**Bug del simulador.** ADDI (encoding `0x0814002A`) genera CAUSE=3 y deja el simulador en estado inválido. Mismo comportamiento que ANDI (bug documentado por el profesor). Instrucción omitida.

---

# Caso 4
## Descripción
`AND` — AND bit a bit.

## Precondiciones
- `$t1` = `0x0000FF0F`
- `$t2` = `0x00FF00FF`

## Code
```asm
AND $t0, $t1, $t2
```

## Postcondiciones
- `$t0` = `0x0000000F`

## Conclusiones
Anduvo. Solo quedaron en 1 los bits comunes a ambos operandos.

---

# Caso 5
## Descripción
`OR` — OR bit a bit.

## Precondiciones
- `$t1` = `0xA0A0A0A0`
- `$t2` = `0x0F0F0F0F`

## Code
```asm
OR $t0, $t1, $t2
```

## Postcondiciones
- `$t0` = `0xAFAFAFAF`

## Conclusiones
Anduvo. La unión de bits produjo `0xAFAFAFAF`.

---

# Caso 6
## Descripción
`XOR` — XOR bit a bit.

## Precondiciones
- `$t1` = `0xDEADBEEF`
- `$t2` = `0xFFFFFFFF`

## Code
```asm
XOR $t0, $t1, $t2
```

## Postcondiciones
- `$t0` = `0x21524110`

## Conclusiones
Anduvo. XOR con `0xFFFFFFFF` invirtió todos los bits.

---

# Caso 7
## Descripción
`NOR` — NOR bit a bit.

## Precondiciones
- `$t1` = `0x0000FFFF`

## Code
```asm
NOR $t0, $t1, $zero
```

## Postcondiciones
- `$t0` = `0xFFFF0000`

## Conclusiones
Anduvo. NOR con `$zero` invirtió todos los bits de `$t1`.

---

# Caso 8
## Descripción
`SLL` — desplazamiento lógico izquierdo.

## Precondiciones
- `$t1` = `0x00000001`

## Code
```asm
SLL $t0, $t1, 3
```

## Postcondiciones
- `$t0` = `0x00000008`

## Conclusiones
Anduvo. 1 << 3 = 8.

---

# Caso 9
## Descripción
`SRL` — desplazamiento lógico derecho (entran ceros, no preserva signo).

## Precondiciones
- `$t1` = `0x80000000`

## Code
```asm
SRL $t0, $t1, 1
```

## Postcondiciones
- `$t0` = `0x40000000`

## Conclusiones
Anduvo. El MSB se desplazó y entró un 0, confirmando desplazamiento lógico.

---

# Caso 10
## Descripción
`SRA` — desplazamiento aritmético derecho (preserva signo).

> **Discrepancia con el manual:** el manual indica `R[rs]` como fuente, pero el hardware usa `R[rt]`. El valor debe estar en rt (r12).

## Precondiciones
- r12 = `0x80000000`

## Code
Machine code: `0x0018A102`
```
[31:27]=00000 R | [26:22]=00000 rs=r0 | [21:17]=01100 rt=r12 | [16:12]=01010 rd=r10 | [11:7]=00010 aux=2 | [5:0]=000010 SRA
```

## Postcondiciones
- r10 = `0xE0000000`

## Conclusiones
Anduvo. SRA propagó el bit de signo correctamente. El manual dice `R[rs]` pero el hardware usa `R[rt]`; con el valor en rs el resultado fue 0, con el valor en rt dio `0xE0000000`.

---

# Caso 11
## Descripción
`SLT` — pone 1 en rd si rs < rt (signed), 0 si no.

## Precondiciones
- `$t1` = `0x00000003`
- `$t2` = `0x00000007`

## Code
```asm
SLT $t0, $t1, $t2
```

## Postcondiciones
- `$t0` = `0x00000001`

## Conclusiones
Anduvo. 3 < 7 dio 1 en `$t0`.

---

# Caso 12
## Descripción
`SW` / `LW` — guardar y cargar una palabra de 32 bits. El simulador solo tiene memoria válida cerca de 0x0000; usar 0x1000 genera CAUSE=2.

## Precondiciones
- r10 = `0xCAFE1234`
- r11 = `0x00000010`

## Code
SW encoding: `0x4AD40000` | LW encoding: `0x42D40000`

## Postcondiciones
- r10 = `0xCAFE1234` (después del LW)

## Conclusiones
Anduvo. SW guardó en `0x00000010` y LW lo recuperó sin alteraciones.

---

# Caso 13
## Descripción
`SW` / `LW` con offset — verifica EA = R[rs] + imm.

## Precondiciones
- r10 = `0xABCD0000`
- r11 = `0x00000010`

## Code
SW encoding: `0x4AD40004` | LW encoding: `0x42D40004`

## Postcondiciones
- r10 = `0xABCD0000` (leído desde `0x00000014`)

## Conclusiones
Anduvo. El offset 4 se sumó a la base sin pisar el dato del Caso 12.

---

# Caso 14
## Descripción
`SB` / `LB` — guardar y cargar un byte con extensión de signo.

## Precondiciones
- r10 = `0x000000FF`
- r11 = `0x00000010`

## Code
SB encoding: `0x5AD40000` | LB encoding: `0x72D40000`

## Postcondiciones
- r10 = `0xFFFFFFFF`

## Conclusiones
Anduvo. SB guardó el byte bajo y LB lo recuperó extendiendo el signo (`0xFF` → `0xFFFFFFFF`).

---

# Caso 15
## Descripción
`LBU` — cargar byte sin extensión de signo.

## Precondiciones
- Byte `0xFF` en `0x00000010` (del Caso 14)
- r11 = `0x00000010`

## Code
Encoding: `0x7AD40000`

## Postcondiciones
- r10 = `0x000000FF`

## Conclusiones
Anduvo. El mismo `0xFF` resultó en 255 en lugar de -1 al no extenderse el signo.

---

# Caso 16
## Descripción
`BEQ` — salta si rs == rt.

## Precondiciones
- Caso con salto: r11 = r12 = `0x00000010`
- Caso sin salto: r11 = `0x00000001`, r12 = `0x00000002`

## Code
Encoding: `0x82D80001`

## Postcondiciones
- Con r11 = r12: PC = `0x00000008`
- Con r11 ≠ r12: PC = `0x00000004`

## Conclusiones
Anduvo. Saltó cuando eran iguales y avanzó normalmente cuando eran distintos.

---

# Caso 17
## Descripción
`BNE` — salta si rs ≠ rt.

## Precondiciones
- Caso con salto: r11 = `0x00000001`, r12 = `0x00000002`
- Caso sin salto: r11 = r12 = `0x00000005`

## Code
Encoding: `0x8AD80001`

## Postcondiciones
- Con r11 ≠ r12: PC = `0x00000008`
- Con r11 = r12: PC = `0x00000004`

## Conclusiones
Anduvo. Saltó cuando eran distintos y no saltó cuando eran iguales.

---

# Caso 18
## Descripción
`BLT` — salta si rs < rt (comparación con signo).

## Precondiciones
- Caso con salto: r11 = `0xFFFFFFFF` (-1), r12 = `0x00000001`
- Caso sin salto: r11 = `0x00000001`, r12 = `0xFFFFFFFF`

## Code
Encoding: `0x92D80001`

## Postcondiciones
- Con r11 < r12 (signed): PC = `0x00000008`
- Con r11 > r12 (signed): PC = `0x00000004`

## Conclusiones
Anduvo. `0xFFFFFFFF` se trató como -1 (signed), confirmando comparación con signo.

---

# Caso 19
## Descripción
`MUL` — multiplicación con signo, guarda los 32 bits bajos.

## Precondiciones
- `$t1` = `0x00000006`
- `$t2` = `0x00000007`

## Code
```asm
MUL $t0, $t1, $t2
```

## Postcondiciones
- `$t0` = `0x0000002A`

## Conclusiones
Anduvo. 6 × 7 = 42 = `0x2A`.

---

# Caso 20
## Descripción
`MULH` — parte alta (bits 63:32) de la multiplicación con signo.

## Precondiciones
- `$t1` = `0x00010000`
- `$t2` = `0x00010000`

## Code
```asm
MULH $t0, $t1, $t2
```

## Postcondiciones
- `$t0` = `0x00000001`

## Conclusiones
Anduvo. 65536² = `0x100000000`; MULH capturó el bit 32 del producto.

---

# Caso 21
## Descripción
`DIV` — división entera con signo, guarda el cociente.

## Precondiciones
- `$t1` = `0x00000014` (20)
- `$t2` = `0x00000004` (4)

## Code
```asm
DIV $t0, $t1, $t2
```

## Postcondiciones
- `$t0` = `0x00000005`

## Conclusiones
Anduvo. 20 ÷ 4 = 5.

---

# Caso 22
## Descripción
`REST` — resto de la división entera.

## Precondiciones
- `$t1` = `0x00000011` (17)
- `$t2` = `0x00000005` (5)

## Code
```asm
REST $t0, $t1, $t2
```

## Postcondiciones
- `$t0` = `0x00000002`

## Conclusiones
Anduvo. 17 mod 5 = 2.

---

# Caso 23
## Descripción
`LUI` — carga inmediato en los 16 bits altos del registro.

## Conclusiones
**Bug del simulador.** LUI (opcode 7) genera CAUSE=3 con h=0 (`0x3814ABCD`) y con h=1 (`0x3815ABCD`). Mismo comportamiento que ADDI. Instrucción omitida.
