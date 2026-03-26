# int-net
Carlos Camacho Soto, marzo 2026

## Redes con flujo intermitente
En países en vías de desarrollo es común que los sistemas de acueductos funciones bajo condiciones de flujo intermitente, 
mayormente debidas a déficit de producción con respecto a la demanda, de modo que la red no es capaz de soportar el patrón
de consumo diario.  Esto ocasiona que las redes puedan entrar a régimen de canal abierto en algunos tramos en determinado
momento del día.  Otras ocasiones, debido a fallas en los elementos productivos, la red puede vaciarce parcial o totalmente
ocasionando desabastecimiento, el cual se prolonga desde el momento de la falla hasta que pueda lograrse la estabilización
del funcionamiento de la red.
El Método del Gradiente Hidráulico (MGH) es una herramienta que permite modelar el funcionamiento de redes bajo régimen de presión,
sin embargo tiende a fallar cuando el caudal aportado a la red no puede satisfacer la demanda.  En estas ocasiones, las líneas
de energía tienden a cortar la topográfica de modo que los nudos tendrán presiones negativas.
Las presiones negativas en el MGH son matemáticamente correctas son físicamente incorrectas, porque la presión mínima que puede
haber en las acometidas de los usuarios es la presión atmosférica, de modo que el caudal en esos nodos no puede ser el que se
se asignó al nodo correspondiente sino que va a tener un valor cero o cercano a cero.
Esta discrepancia entre los caudales asignados y los posibles, introduce un valor de caudal asociado a los tramos conectados
a esos nodos que no representa las posibilidades físicas, es decir, que esos caudales estarán sobreestimados.
Se entra así en un círculo vicioso en el que las pérdidas de carga asignadas al tramo también están sobreestimadas y la 
piezométrica no se clavará tanto y la presión en el nodo no será negativa.
Reponer al sistema de este funcionamiento con nodo negativo es necesario para encontrar el verdadero equilibrio entre los caudales
en tramos, demandas en (mejor dicho aportes a) los nudos y corregir el cálculo de las pérdidas de energía en cada tramo, ya sea
funcionando a presión como a canal abierto es necesario para poder representar la realidad del funcionamiento de la red.
Del análisis de las presiones negativas se puede inferir cuales puntos de la red van a tener problemas de funcionamiento, sin
embargo esas presiones negativas están teñidas de la incertidumbre debida a que el equilibrio puede ser otro.

## Objetivos del proyecto
* Desarrollo de un modelo de redes hidráulicas de redes intermitentes usando el Método de Gradiente Hidráulico (EPAnet) y rutinas
o aplicaciones que permitan corregir las corridas del modelo de EPAnet cuando los nudos lleguen a trabajar con presiones negativas.
* Corregir los valores de presiones negativas en la red mediante un proceso de disminución de caudales en los nodos y la 
consecuente variación de la línea piezométrica asociada.  Patirana et al. proponen el uso de los emisores con valores negativos
para lograr cambiar el caudal demandado en los nudos, usando demandas dependientes de la presión.  Sin embargo esta solución implica
que la red no entra en régimen a canal abierto.
* Definir un algoritmo para modificar los archivos de entrada (.INP ) de EPAnet para realizar los cambios en los valores de la demanda
de los nudos con presiones negativas, hasta llegar a un equilibrio.  Esto va a reemplazar la curva de demanda teórica con una curva de
demanda posible.  Esto implica una curva de demanda para cada nodo ó grupo de nodos.
* Cuando se llega al equilibrio comparar los valores de las demandas iniciales y los caudales aportados a cada nodo para determinar 
un déficit de abastecimiento.
* Determinar los tiempos de llenado y vaciado del sistema, a manera de aproximación usando flujo permanente.  Puede buscarse una metodología
para modelar flujo no permanente.
* Investigar la forma de modelar el sistema mientras se realiza el llenado y

## Ideas de desarrollo
* Propagación del llenado  (y posiblemente de vaciado en sentido opuesto): 
** Se inicia con el punto más bajo, a este se le asigna un tiempo inicial [t_0] que resulte en la distancia más corta entre la fuente y el 
nudo, dividida entre la velocidad media del flujo en la red.  Puede usarse un valor inicial de 2[m/s]
** Se hace un barrido de los nudos desde abajo hacia arriba, calculando el tiempo de llenado de cada tubería conectada al nudo con
la longitud de la tubería dividida entre la velocidad del flujo media: [t_i = L/v].   
** Se le asigna a cada nodo siguiente el tiempo del nudo anterior más el tiempo del tramo calculado como se indicó.  Si un nudo está 
conectado a varior tubos, el tiempo del nudo será el que resulte ser el menor de las posibilidades dadas.
marcan los tubos llenado así.   
** Luego se continúa con el siguiente nudo con la elevación menor de los que quedan, calculando el tiempo del flujo en cada tubo que no
haya sido considerado antes. Se asignan los tiempos y se continúa con el proximo.
** El proceso acaba cuando se llegue el último nodo, generalmente el nudo de elevación fija ya sea tanque o reservorio.
** En el proceso se asigna un caudal en cada nudo incrementalmente de abajo hacia arriba, asignando el caudal de los tramos tributarios al
nudo al final de cada trramo, es decir en el nudo.  Luego se debe descontar este caudal del demandado en cada nudo, ya que no es 
posible satisfacer las 2 demandas: demanda del usuario y demanda de llenado.  Estos valores de caudal son los que pueden usarse para
modelar el funcionamienro de la red mientras no llena.
A cada paso de un tiempo DeltaT, los nudos van entrando a presión y el valor del caudal demandado empieza a estabilizarse de abajo hacia 
arriba.
* Usar EPANet en línea de comando y hacer los algoritmos y rutinas de modificación de los archivos de entrada y análisis de los archivos 
de salida en BASH, Python y otras herramientas de procesamiento de texto.



