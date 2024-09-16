# Команды

## Арифметические команды

| Название команды | Описание                                                                         | Пример использования  | Реализация             |
| ---------------- | -------------------------------------------------------------------------------- | --------------------- | ---------------------- |
| `MOV`            | Копирует значение из одного регистра в другой                                    | `MOV R_dst, R_src`    | [MOV инструкция](#mov) |
| `DEC`            | Уменьшает значение регистра на `1`                                               | `DEC R_dst`           | [DEC инструкция](#dec) |
| `ADD`            | Складывает значения двух регистров и сохраняет результат в третий регистр        | `ADD R_a, R_b, R_res` | [ADD инструкция](#add) |
| `SUB`            | Выполняет вычитание значения регистра из значения регистра и сохраняет результат | `SUB R_a, R_b, R_res` | [SUB инструкция](#sub) |
| `MUL`            | Умножает значения двух регистров и сохраняет результат в третий регистр          | `MUL R_a, R_b, R_res` | [MUL инструкция](#mul) |
| `DIV`            | Делит значение одного регистра на значение другого, результат сохраняется        | `DIV R_a, R_b, R_res` | [DIV инструкция](#div) |
| `MOD`            | Вычисляет остаток от деления значений двух регистров, результат сохраняется      | `MOD R_a, R_b, R_res` | [MOD инструкция](#mod) |

### MOV

Копирует значение из одного регистра (`R_src`) в другой (`R_dst`).

Математическое представление: $(R_{\text{dst}} \leftarrow R_{\text{src}} \text{ ; } \text{IP} = \text{IP} + 1)$

```C
MOV R_dst, R_src
```

```C
ZERO R_dst                // Обнуляем целевой регистр
BRAN R_src, R_dst, done   // Если R_src == R_dst, завершаем

loop:
  INCR R_dst              // Увеличиваем R_dst
  BRAN R_src, R_dst, done // Если R_src == R_dst, завершить
  JMP loop                // Иначе продолжаем увеличивать

done:
STOP
```

[См. реализацию команды JMP](#jmp).

### DEC

Уменьшает значение регистра (`R_dst`) на `1`.

Математическое представление: $(R_\text{dst} \leftarrow R_\text{dst} - 1 \text{ ; } \text{IP} = \text{IP} + 1)$

```C
DEC R_dst
```

```C
ZERO R_temp                     // Обнуляем вспомогательные регистры
ZERO R_temp_next
INCR R_temp_next
BRAN R_dst, R_temp, done        // Если R_dst == R_temp, завершаем
BRAN R_dst, R_temp_next, done   // Если R_dst == R_temp_next, завершаем

loop:
  INCR R_temp                   // Увеличиваем вспомогательные регистры
  INCR R_temp_next
  BRAN R_dst, R_temp_next, done // Если R_dst == R_temp, завершить
  JMP loop                      // Повторяем, пока не достигнем уменьшения

done:
STOP
```

[См. реализацию команды JMP](#jmp).

### ADD

Складывает значения двух регистров (`R_a` и `R_b`) и сохраняет результат в третий регистр (`R_res`).

Математическое представление: $(R_{\text{res}} \leftarrow R_a + R_b \text{ ; } \text{IP} = \text{IP} + 1)$

```C
ADD R_a, R_b, R_res
```

```C
ZERO R_res                  // Обнуляем регистр результата
ZERO R_temp                 // Обнуляем регистр суммы с регистром
BRAN R_temp, R_a, next      // Если R_temp == R_a, переходим ко второму операнду

loop_a:
  INCR R_res                // Увеличиваем результат на значение R_a
  INCR R_temp
  BRAN R_a, R_temp, next    // Если R_a == R_temp, переход
  JMP loop_a

next:
ZERO R_temp                 // Обнуляем регистр суммы
BRAN R_temp, R_b, done      // Если R_temp == R_b, выход

loop_b:
  INCR R_res                // Увеличиваем результат на значение R_b
  INCR R_temp
  BRAN R_b, R_temp, done    // Если R_b == R_temp, выход
  JMP loop_b

done:
STOP
```

[См. реализацию команды JMP](#jmp).

### SUB

Выполняет вычитание значения регистра `R_b` из значения регистра `R_a` и сохраняет результат в `R_res`

Математическое представление: $(R_{\text{res}} \leftarrow R_a - R_b \text{ ; } \text{IP} = \text{IP} + 1)$

```C
SUB R_a, R_b, R_res
```

```C
MOV R_res R_a          // R_res = R_a
MOV R_temp R_b         // R_temp = R_b
BRAN R_temp, 0, done    // Если R_temp == 0, выход

loop:
  DEC R_res             // Увеличиваем результат на значение R_a
  DEC R_temp
  BRAN R_temp, 0, done  // Если R_temp == 0, выход
  JMP loop

done:
STOP
```

[См. реализацию команды MOV](#mov).

[См. реализацию команды JMP](#jmp).

[См. реализацию команды DEC](#dec).

### MUL

Умножает значения двух регистров (`R_a` и `R_b`) и сохраняет результат в третий регистр (`R_res`).

Математическое представление: $(R_{\text{res}} \leftarrow R_a * R_b; \text{IP} = \text{IP} + 1)$

```C
MUL R_a, R_b, R_res
```

```C
ZERO R_res                // Обнуляем регистр результата
BRAN R_a, 0, done         // Если R_a == 0, выход
BRAN R_b, 0, done         // Если R_b == 0, выход
ZERO R_temp

loop:
  INC R_temp
  ADD R_res R_a R_a
  BRAN R_temp, R_b, done  // Если R_temp == R_b, выход
  JMP loop

done:
STOP
```

[См. реализацию команды JMP](#jmp).

[См. реализацию команды ADD](#add).

### DIV

Делит значение одного регистра (`R_a`) на значение другого (`R_b`), результат сохраняется (`R_res`).

Математическое представление: $(R_{\text{res}} \leftarrow \left\lfloor \frac{R_a}{R_b} \right\rfloor; \text{IP} = \text{IP} + 1)$

```C
DIV R_a, R_b, R_res
```

```C
ZERO R_res            // Обнуляем регистр результата
BRAN R_b, 0, done     // Если R_b == 0, завершить
BRAN R_a, 0, done     // Если R_a == 0, завершить

loop:
  SUB R_a, R_a, R_b   // Вычитаем делитель из делимого
  INCR R_res          // Увеличиваем результат
  JG R_a, R_b, loop   // Если R_a >= R_b, продолжаем цикл

done:
STOP
```

[См. реализацию команды SUB](#sub).

[См. реализацию команды JG](#jg).

### MOD

Вычисляет остаток от деления значений двух регистров (`R_a` и `R_b`), результат сохраняется (`R_res`).

Математическое представление: $(R_{\text{res}} \leftarrow R_a \mod R_b; \text{IP} = \text{IP} + 1)$

```C
MOD R_a, R_b, R_res
```

```C
ZERO R_res            // Обнуляем регистр результата
BRAN R_a, R_b, done   // Если R_a == R_b, завершить (остаток 0)

loop:
  SUB R_a, R_a, R_b   // Вычитаем делитель из делимого
  JG R_a, R_b, loop // Если R_a >= R_b, продолжаем

MOV R_res R_a

done:
STOP
```

[См. реализацию команды MOV](#mov).

[См. реализацию команды SUB](#sub).

[См. реализацию команды JG](#jg).

## Переходы

| **Название команды** | **Описание**                                                     | **Пример использования** | **Реализация**         |
| -------------------- | ---------------------------------------------------------------- | ------------------------ | ---------------------- |
| `JMP`                | Безусловный переход на указанную метку или адрес                 | `JMP target`             | [JMP инструкция](#jmp) |
| `JNE`                | Переход, если значения двух регистров не равны                   | `JNE R_a, R_b, target`   | [JNE инструкция](#jne) |
| `JG`                 | Переход, если значение первого регистра больше второго           | `JG R_a, R_b, target`    | [JG инструкция](#jg)   |
| `JGE`                | Переход, если значение первого регистра больше или равно второму | `JGE R_a, R_b, target`   | [JGE инструкция](#jge) |
| `JL`                 | Переход, если значение первого регистра меньше второго           | `JL R_a, R_b, target`    | [JL инструкция](#jl)   |
| `JLE`                | Переход, если значение первого регистра меньше или равно второму | `JLE R_a, R_b, target`   | [JLE инструкция](#jle) |
| `JZ`                 | Переход, если значение регистра равно нулю                       | `JZ R_dst`               | [JZ инструкция](#jz)   |
| `JNZ`                | Переход, если значение регистра не равно нулю                    | `JNZ R_dst`              | [JNZ инструкция](#jnz) |

### JMP

Безусловный переход на указанную метку или адрес (`target`)

Математическое представление: $(\text{IP} = \text{target})$

```C
JMP target
```

```C
BRAN R_x, R_x, target
```

### JNE

Переход (`target`), если значения двух регистров (`R_a` и `R_b`) не равны.

Математическое представление: $[R_a \neq R_b \text{ ? } \text{IP} = \text{target} : \text{IP} = \text{IP} + 1]$

```C
JNE R_a, R_b, target
```

```C
BRAN R_a, R_b, done
JMP target

done:
STOP
```

[См. реализацию команды JMP](#jmp).

### JG

Переход (`target`), если значение первого регистра (`R_a`) больше второго (`R_b`).

Математическое представление: $[R_a > R_b \text{ ? } \text{IP} = \text{target} : \text{IP} = \text{IP} + 1]$

```C
JG R_a, R_b, target
```

```C
BRAN R_a, R_b, done   // Завершаем если R_a == R_b

loop:
  DEC R_a             // Уменьшаем R_a
  DEC R_b             // Уменьшаем R_b
  BRAN R_a, 0, done   // Если R_a == 0 (то есть R_a < R_b), завершаем
  BRAN R_b, 0, target // Если R_b == 0 (то есть R_a > R_b), переход
  JMP loop            // Иначе продолжаем цикл

done:
STOP
```

[См. реализацию команды JMP](#jmp).

[См. реализацию команды DEC](#dec).

### JGE

Переход (`target`), если значение первого регистра (`R_a`) больше или равно второму (`R_b`).

Математическое представление: $[R_a \geq R_b \text{ ? } \text{IP} = \text{target} : \text{IP} = \text{IP} + 1]$

```C
JGE R_a, R_b, target
```

```C
BRAN R_a, R_b, target  // Переход если R_a == R_b

loop:
  DEC R_a              // Уменьшаем R_a
  DEC R_b              // Уменьшаем R_b
  BRAN R_a, 0, done    // Если R_a == 0 (то есть R_a < R_b), завершаем
  BRAN R_b, 0, target  // Если R_b == 0 (то есть R_a > R_b), переход
  JMP loop             // Иначе продолжаем цикл

done:
STOP
```

[См. реализацию команды JMP](#jmp).

[См. реализацию команды DEC](#dec).

### JL

Переход (`target`), если значение первого регистра (`R_a`) меньше второго (`R_b`).

Математическое представление: $[R_a < R_b \text{ ? } \text{IP} = \text{target} : \text{IP} = \text{IP} + 1]$

```C
JL R_a, R_b, target
```

```C
BRAN R_a, R_b, done    // Завершаем если R_a == R_b

loop:
  DEC R_a              // Уменьшаем R_a
  DEC R_b              // Уменьшаем R_b
  BRAN R_b, 0, done    // Если R_b == 0 (то есть R_a > R_b), завершаем
  BRAN R_a, 0, target  // Если R_a == 0 (то есть R_a < R_b), переход
  JMP loop             // Иначе продолжаем цикл

done:
STOP
```

[См. реализацию команды JMP](#jmp).

[См. реализацию команды DEC](#dec).

### JLE

Переход (`target`), если значение первого регистра (`R_a`) меньше или равно второму (`R_b`).

Математическое представление: $[R_a \leq R_b \text{ ? } \text{IP} = \text{target} : \text{IP} = \text{IP} + 1]$

```C
JLE R_a, R_b, target
```

```C
BRAN R_a, R_b, target  // Переходим если R_a == R_b

loop:
  DEC R_a              // Уменьшаем R_a
  DEC R_b              // Уменьшаем R_b
  BRAN R_b, 0, done    // Если R_b == 0 (то есть R_a > R_b), завершаем
  BRAN R_a, 0, target  // Если R_a == 0 (то есть R_a < R_b), переход
  JMP loop             // Иначе продолжаем цикл

done:
STOP
```

[См. реализацию команды JMP](#jmp).

[См. реализацию команды DEC](#dec).

### JZ

Переход (`target`), если значение регистра (`R_dst`) равно нулю.

Математическое представление: $[R_\text{ dst } = 0 \text{ ? } \text{IP} = \text{target} : \text{IP} = \text{IP} + 1]$

```C
JZ R_dst, target
```

```C
BRAN R_dst, 0, target  // Переходим если R_dst == 0
```

### JNZ

Переход (`target`), если значение регистра (`R_dst`) не равно нулю.

Математическое представление: $[R_\text{ dst } \neq 0 \text{ ? } \text{IP} = \text{target} : \text{IP} = \text{IP} + 1]$

```C
JNZ R_dst, target
```

```C
BRAN R_dst, 0, done  // Завершаем если R_dst == 0
JMP target

done:
STOP
```

[См. реализацию команды JMP](#jmp).
