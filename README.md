punto 1

Diseño basado en Concurrencia y Cálculo PI

1.1 Conceptos esenciales del π-calculus

El π-calculus es un marco formal para modelar sistemas concurrentes que se comunican mediante paso de mensajes en canales. Permite representar escenarios distribuidos donde la estructura de comunicación puede cambiar durante la ejecución, ya que los nombres de canales pueden enviarse entre procesos.

Principales propiedades:
	•	Concurrencia (P | Q): Dos procesos operan al mismo tiempo.
	•	Recepción c(x).P: Un proceso permanece a la espera de recibir un valor en el canal c.
	•	Envío : Manda un valor y por el canal c.
	•	Replicación (!P): Genera infinitas copias del proceso P.
	•	Restricción (ν x)P: Crea un canal privado x.
	•	Nil (0): Representa un proceso finalizado

  1.3 Canales de Comunicación

Los canales facilitan la sincronización y transferencia de datos entre procesos:
	•	params_channel: Envía los parámetros actuales
	•	pred_channel: Manda las predicciones
	•	gradient_channel: Entrega los gradientes calculados
	•	update_channel: Informa actualizaciones
	•	control_channel: Coordina la ejecución entre épocas
	•	error_channel: Reporta el MSE

⸻

1.4 Especificación del Sistema

La estructura se expresa de la siguiente manera:

Master = (ν params_ch, pred_ch, grad_ch, upd_ch).
params_ch⟨w_init, b_init⟩ |
!Epoch(params_ch, pred_ch, grad_ch, upd_ch, epoch_count)
Epoch(params, pred_ch, grad_ch, upd_ch, count) =
if count < MAX_EPOCHS then
params(w, b).
Predictor(pred_ch) | GradientCalc(grad_ch) |
ParamUpdater(upd_ch) |
pred_ch⟨w, b, X, y⟩.
pred_ch(y_pred).
grad_ch⟨y_pred, y⟩.
grad_ch(dw, db).
upd_ch⟨w, b, dw, db, lr⟩.
upd_ch(w_new, b_new).
params⟨w_new, b_new⟩ |
Epoch(params, count+1)
else
params(w_final, b_final).
output⟨w_final, b_final⟩
Predictor(pred_ch) =
pred_ch(w, b, X, y).
let y_pred = w * X + b in
pred_ch⟨y_pred⟩
GradientCalc(grad_ch) =
grad_ch(y_pred, y).
let error = y_pred - y in
let dw = (2/m) * sum(error * X) in
let db = (2/m) * sum(error) in
grad_ch⟨dw, db⟩
ParamUpdater(upd_ch) =
upd_ch(w, b, dw, db, lr).
let w_new = w - lr * dw in
let b_new = b - lr * db in
upd_ch⟨w_new, b_new⟩


1.5 Estrategia de Paralelización
Nivel 1 - Paralelización de Datos
Dividir el dataset X, y en N particiones y calcular gradientes parciales en paralelo:
(ν partition1
_
ch, ..., partitionN
_
ch).
(Partition1(partition1
_
ch) |
Partition2(partition2
_
ch) |
...
PartitionN(partitionN
_
ch)) |
GradientAggregator(partition1
_
ch, ..., partitionN
_
ch)
Nivel 2 - Pipeline de Épocas
Mientras una época calcula gradientes, otra puede actualizar parámetros en paralelo:
Epoch
_
i(Predict) | Epoch
_
i(CalcGrad) | Epoch
_{i-1}(Update)
Barrera de Sincronización
Implementar barreras para coordinar múltiples procesos:
Barrier(sync
_
ch, n
_
workers) =
let count = ref 0 in
!sync
_
ch(worker
_
id).
count := !count + 1 |
if !count == n
workers then
_
broadcast⟨continue⟩ |
count := 0
1.6 Ventajas del Diseño Concurrente
•
•
•
•
•
Escalabilidad: Cada proceso ejecutable en núcleos o máquinas diferentes
Modularidad: Separación clara de responsabilidades entre procesos
Dinamismo: Canales creados dinámicamente para cada época
Tolerancia a fallos: Procesos pueden replicarse usando
Composicionalidad: Procesos se pueden combinar de formas nuevas
