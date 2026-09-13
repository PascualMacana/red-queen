# reinante

Un programa chico en Rust que reescribe un pedazo de su propia fuente mientras **el objetivo mismo se sigue moviendo**.

No es un modelo de lenguaje y no se copia solo. Le señalás una carpeta; sólo escribe ahí.

Es hermano de [mejorante](https://github.com/PascualMacana/mejorante). Darwin sigue proponiendo (mutantes al azar, el mismo lenguaje prefijo). La Reina Roja es el mundo: cuando el cerebro encaja, la función de puntaje muta y el encaje se cae. Hay que seguir corriendo para quedarse en el mismo lugar.

Lo que evoluciona es una expresión matemática chica, el **cerebro**. Intenta encajar otra expresión, el **mundo**, en `x = -5 … 5`. El puntaje es la suma de errores al cuadrado (**sse**). Más bajo es mejor. Cero es un alcance — y entonces el mundo salta.

```
padre     cerebro  0                              mundo  x² + 3x + 5     sse  4323
  │ evolve
  │ catch
  ▼
          cerebro  (+ (+ (* 3 x) (* x x)) 5)      mundo  x² + 3x + 5     sse  0
  │ el mundo muta
  ▼
hijo      cerebro  (+ (+ (* 3 x) (* x x)) 5)      mundo  (se movió)      sse  > 0
```

El primer mundo es el mismo polinomio que los hermanos congelan. Después de un alcance, ya no.

![Una célula que se llena y se vacía cuando el mundo salta](cell.svg)

Míralo en la terminal. Una célula que se llena mientras baja el error. Cuando alcanza, la curva objetivo se mueve y el cuerpo se vacía. `dish` usa por defecto la seed 7, que alcanza el mundo de apertura en pocos pasos y después tiene que volver a perseguir.

```bash
cargo run -- dish
```

## Cómo correrlo

Hace falta [Rust](https://rustup.rs/).

```bash
cargo build --release
./target/release/reinante identity
./target/release/reinante evolve --steps 120 --spawn ./hijo --build
./hijo/target/debug/reinante identity
```

`identity` imprime generación, cerebro, mundo, huidas y puntaje.  
`evolve --spawn ./hijo --build` busca, deja que el mundo huya al ser alcanzado, y escribe un crate hijo que hereda cerebro y mundo.

## Comandos

```
reinante identity              generación, linaje, cerebro, mundo, huidas, puntaje
reinante eval [x]              cerebro vs el mundo actual
reinante evolve                busca; el mundo huye al ser alcanzado
                 --steps N      pasos de búsqueda (default 120)
                 --lambda L     mutantes por paso (default 30)
                 --seed S       rng reproducible
                 --spawn <dir>  hijo con el cerebro y el mundo del final
                 --build        compila a ese hijo
                 --force        pisa un hijo anterior
                 --write        pisa src/main.rs de este proyecto
reinante dish                  anima una célula que se llena y se vacía
                 --steps N      pasos de búsqueda (default 120)
                 --lambda L     mutantes por paso (default 30)
                 --seed S       default 7 (la demo fiable)
                 --delay MS     ms por cuadro (default 80)
reinante spawn <dir>           copia el genoma actual (sin buscar)
reinante genome                imprime las fuentes embebidas
```

`--spawn` deja este programa en paz y escribe un hijo elegido.  
`--write` edita el `src/main.rs` de este proyecto; compilá de nuevo para que el binario nazca con el cerebro y el mundo nuevos.

## Cómo funciona

El cerebro y el mundo viven como expresiones prefijas en `src/main.rs`: `x`, enteros chicos, `+`, `-`, `*`.

1. Parsea el cerebro y el mundo actuales a árboles.
2. Cada paso arma varios mutantes del cerebro. Se queda con el menor `sse + 0.01 × tamaño` contra el mundo **actual**.
3. Después de un encaje perfecto, las identidades algebraicas pueden achicar el cerebro.
4. Entonces el mundo muta. Si el cerebro deja de ser perfecto, esa huida cuenta.
5. Con `--spawn`, escribe un proyecto Cargo completo cuya fuente contiene ese cerebro **y** ese mundo.

No hay techo congelado. Un hijo nace en el mundo que estaba persiguiendo, no en `x² + 3x + 5` para siempre.

Esto no es la Gödel Machine de Schmidhuber y no es una prueba: no hay certificado. Eso está en [demostrante](https://github.com/PascualMacana/demostrante). Los hermanos congelan el evaluador; este no.

## Seguridad

- Un hijo por corrida. No hay bucles en segundo plano ni red.
- No escribe sobre el directorio home, `/`, `/usr`, `/etc`, ni el directorio en el que estás parado.
- `--force` sólo borra una carpeta que ya parece un proyecto `reinante`.

## Relacionados

[replicante](https://github.com/PascualMacana/replicante) se copia.  
[mejorante](https://github.com/PascualMacana/mejorante) se copia y además intenta mejorar, contra un objetivo congelado.  
[demostrante](https://github.com/PascualMacana/demostrante) sólo escribe una mejora afirmada si hay una prueba verificable.  
[cruzante](https://github.com/PascualMacana/cruzante) se queda con los cruces del río que todavía eran legales.
