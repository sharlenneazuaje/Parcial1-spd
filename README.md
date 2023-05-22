## Alumna: Sharlenne Azuaje


## Proyecto: Montacarga Funcional.
![Tinkercad](./Img/Montacarga2.png)


## Descripción
Modelo de montacarga funcional como maqueta para un hospital. 
Sistema que puede recibir ordenes de subir, bajar o pausar desde diferentes pisos y muestra el estado actual del montacargas en el display 7
segmentos.


## Lista de Componentes
![Tinkercad](./Img/ListadeComponentesMontacarga2.png)


## Diagrama Esquemático del Circuito
![Tinkercad](./Img/DE21.png)
![Tinkercad](./Img/DE22.png)

- Arduino Uno: nos permite interactuar con el display, los leds y los botones. El Arduino lee el estado de los botones y envía las señales adecuadas a los leds y el display. 
- Botones Pull-Down: tenemos tres botones, uno destinado a subir, otro a bajar y otro a pausar. Al apretar el botón subir, el led verde se enciende y en el display se muestra el número del piso actual. Al apretar el botón pausar, el led verde se apaga, el led rojo se enciende y el display muestra el número del piso actual en el que se ha pausado el montacarga. Al apretar el botón bajar, el led verde se enciende y el led rojo se apaga, y en el display se va mostrando el número del piso actual.
- Leds: emiten una luz cuando se aplica corriente eléctrica a través de ellos. En este caso, se encienden o se apagan cuando se pulsan determinados botones. El led verde se enciende cuando el sistema está en movimiento, led rojo se enciende cuando el sistema está pausado y el led azul se enciende para indicar que es de noche. 
- Display 7 segmentos: proporciona una salida visual de información, en este caso mostrando los números desde el 0 (piso más bajo) al 9 (piso más alto).
- Resistencias: nos ayudan a controlar la cantidad de corriente que pasa por los componentes evitando así daños debido a corrientes excesivas. 
- Servo: tenemos un servo que según el estado del sistema, se moverá a una determinada posición. Sistema pausado, 90 grados; sistema en movimiento, 180 grados. 
- Sensor de luz ambiental: miden la intensidad de la luz presente en el entorno. En este caso si es de noche, el led azul se encenderá para indicarlo, caso contrario, no se encenderá.
- Sensor de inclinación: detecta y mide el ángulo de inclinación de un objeto o superficie. En este caso, si el sensor está inclinado, la luz indicativa de la noche se encenderá, caso contrario, no lo hará.


## Funciones 

- DISPLAY: 
LED_A, LED_B, LED_C, LED_D, LED_E, LED_F, LED_F son #define que utilizamos para agregar los pines del display.

- BOTONES: 
BOTON_SUBIR, BOTON_BAJAR, BOTON_PAUSAR son #define que utilizamos para agregar los pines de los botones.

- LEDS: 
LED_ROJO, LED_VERDE, LED_AZUL son #define que utilizamos para agregar los pines de los leds. 

- ESTADOS: 
DETENIDO, SUBIENDO, BAJANDO son #define que utilizamos para establecer los estados del sistema. 

- SERVO:
SERVO1 es #define que utilizamos para agregar el pin del servo.

- SENSOR DE LUZ AMBIENTAL:
SLUZAMBIENTAL es #define que utilizamos para agregar el pin del sensor.

- SENSOR DE INCLINACIÓN: 
SINCLINACION es #define que utilizamos para agregar el pin del sensor.


- VARIABLES GLOBALES: 
int piso_actual; -> variable que guarda el piso actual en el que se encuentra el montacarga
int lectura_subir; -> variable que lee el botón subir
int lectura_bajar; -> variable que lee el botón bajar
int lectura_pausar; -> variable que lee el botón pausar
int habilitacion; -> variable tipo bandera que permite identificar en todo momento si el usuario desea pausar el montacarga
int estadoMontacargas; -> variable que guarda el estado del montacarga dependiendo de la ejecución
int valorSensorLuz; -> variable que guarda el valor que devuelve la lectura del sensor
int valorInclinacion; -> variable que guarda el valor que devuelve la lectura del sensor

- Esta función corresponde al loop en donde se llaman a las demás funciones que se desarrollan. 
~~~ C 
}

void loop(){ 
  
  MostrarNumeroPorDisplay(piso_actual);
  
  if(estadoMontacargas == DETENIDO){ 
        PrenderLuzRoja();
    
        MoverServo90();
    
        EvaluarDiaNocheInclinacion();
    
        lectura_bajar=digitalRead(BOTON_BAJAR);
        lectura_subir=digitalRead(BOTON_SUBIR);
        
        if(lectura_bajar == HIGH || lectura_subir == HIGH){
                if(lectura_subir==HIGH){
                    if(piso_actual < 9){
                      	Serial.println("SUBIENDO...");
                        estadoMontacargas=SUBIENDO;
                    }
                }
                
                if(lectura_bajar==HIGH){ 
                    if(piso_actual > 0){
                        Serial.println("BAJANDO...");
                        estadoMontacargas=BAJANDO;
                    }
                }
        }
  }
  else{ //Está subiendo o bajando
    PrenderLuzVerde();
    
    MoverServo180();
    
    EvaluarDiaNocheInclinacion();
    
    if(estadoMontacargas == SUBIENDO){
            if(piso_actual < 9){
              	Demora();
              	if(habilitacion==1){
                 	IncrementarPiso();
                  	Serial.println("Usted se encuentra en piso: ");  
  					Serial.println(piso_actual);
                    Serial.println("Servo en 180");
              	}
              	else{
                	estadoMontacargas=DETENIDO;
                  	Serial.println("SISTEMA PAUSADO");
                    Serial.println("Servo en 90");
              	}
            }
            else{
                estadoMontacargas=DETENIDO;
            }
    }
    else{ //si estadoMontacargas = bajando
            if(piso_actual > 0){
              	Demora();
              	if(habilitacion==1){
                 	DecrementarPiso();
                  	Serial.println("Usted se encuentra en piso: ");  
  					Serial.println(piso_actual);
                    Serial.println("Servo en 180");
              	}
              	else{
                	estadoMontacargas=DETENIDO;
                  	Serial.println("SISTEMA PAUSADO");
                    Serial.println("Servo en 90");
              	}
            }
            else{
                estadoMontacargas = DETENIDO;
            }
        
    }
  } 
}

~~~

- Esta función mueve el servo a 90 grados.
~~~ C 
void MoverServo90()
{
  myservo1.write(90);
}
~~~

- Esta función mueve el servo a 180 grados.
~~~ C 
void MoverServo180()
{
  myservo1.write(180);
}
~~~

- Esta función evalúa si es de día o de noche y prende el led azul en caso de ser de noche.
~~~ C 
void EvaluarDiaNocheInclinacion()
{
  valorSensorLuz = analogRead(SLUZAMBIENTAL);
  valorInclinacion = analogRead(SINCLINACION);
  Serial.println(valorInclinacion);
    if(valorSensorLuz > 20 && valorInclinacion == 1023)
    {
      digitalWrite(LED_AZUL, HIGH);
    }
    else
    {
      digitalWrite(LED_AZUL, LOW);
    }
  
}
~~~

- Esta función incrementa en 1 a la variable global piso_actual.
~~~ C 
void IncrementarPiso(){
    piso_actual++;
}
~~~


- Esta función decrementa en 1 a la variable global piso_actual.
~~~ C 
void DecrementarPiso(){
    piso_actual--;
}
~~~


- Esta función permite identificar el momento en el que el 
usuario decide pausar el sistema.
~~~ C 
void Demora(void){
    habilitacion=1;
    for(int i=0;i<30;i++){
        delay(100);
        lectura_pausar=digitalRead(BOTON_PAUSAR);
        if(lectura_pausar==HIGH){
            habilitacion=0;
            break;
        }
    }
}
~~~


- Esta función apaga el led verde y prende el led rojo.
~~~ C 
void PrenderLuzVerde(){
       digitalWrite(LED_VERDE, HIGH);
       digitalWrite(LED_ROJO, LOW);
}
~~~


- Esta función prende el led verde y apaga el led rojo.
~~~ C 
void PrenderLuzRoja(){
       digitalWrite(LED_VERDE, LOW);
       digitalWrite(LED_ROJO, HIGH);
}
~~~


- Esta función muestra por el display el número correspondiente al número que se le pasa como parámetro.
~~~ C 
void MostrarNumeroPorDisplay(int piso){
   switch(piso)
   {
	case 0:
        digitalWrite(LED_A, HIGH);      
        digitalWrite(LED_B, HIGH); 
        digitalWrite(LED_C, HIGH);
        digitalWrite(LED_D, HIGH);
        digitalWrite(LED_E, HIGH);
        digitalWrite(LED_F, HIGH);
        digitalWrite(LED_G, LOW); 
	    break;
    case 1:
        digitalWrite(LED_A, LOW);      
        digitalWrite(LED_B, HIGH); 
        digitalWrite(LED_C, HIGH);
        digitalWrite(LED_D, LOW);
        digitalWrite(LED_E, LOW);
        digitalWrite(LED_F, LOW);
        digitalWrite(LED_G, LOW); 
	    break;
    case 2:
 	digitalWrite(LED_A, HIGH);      
        digitalWrite(LED_B, HIGH); 
        digitalWrite(LED_C, LOW);
        digitalWrite(LED_D, HIGH);
        digitalWrite(LED_E, HIGH);
        digitalWrite(LED_F, LOW);
        digitalWrite(LED_G, HIGH); 
        break;
	case 3: 
	digitalWrite(LED_A, HIGH);      
        digitalWrite(LED_B, HIGH); 
        digitalWrite(LED_C, HIGH);
        digitalWrite(LED_D, HIGH);
        digitalWrite(LED_E, LOW);
        digitalWrite(LED_F, LOW);
        digitalWrite(LED_G, HIGH); 
	  break;
	case 4:
        digitalWrite(LED_A, LOW);      
        digitalWrite(LED_B, HIGH); 
        digitalWrite(LED_C, HIGH);
        digitalWrite(LED_D, LOW);
        digitalWrite(LED_E, LOW);
        digitalWrite(LED_F, HIGH);
        digitalWrite(LED_G, HIGH); 
        break;
    case 5:
	digitalWrite(LED_A, HIGH);      
        digitalWrite(LED_B, LOW); 
        digitalWrite(LED_C, HIGH);
        digitalWrite(LED_D, HIGH);
        digitalWrite(LED_E, LOW);
        digitalWrite(LED_F, HIGH);
        digitalWrite(LED_G, HIGH); 
	  break;
	case 6:
	digitalWrite(LED_A, HIGH);      
        digitalWrite(LED_B, LOW); 
        digitalWrite(LED_C, HIGH);
        digitalWrite(LED_D, HIGH);
        digitalWrite(LED_E, HIGH);
        digitalWrite(LED_F, HIGH);
        digitalWrite(LED_G, HIGH); 
     	break;
	case 7:
	digitalWrite(LED_A, HIGH);      
        digitalWrite(LED_B, HIGH); 
        digitalWrite(LED_C, HIGH);
        digitalWrite(LED_D, LOW);
        digitalWrite(LED_E, LOW);
        digitalWrite(LED_F, LOW);
        digitalWrite(LED_G, LOW); 
	  break;
	case 8:
	digitalWrite(LED_A, HIGH);      
        digitalWrite(LED_B, HIGH); 
        digitalWrite(LED_C, HIGH);
        digitalWrite(LED_D, HIGH);
        digitalWrite(LED_E, HIGH);
        digitalWrite(LED_F, HIGH);
        digitalWrite(LED_G, HIGH); 
        break;
    case 9:
	digitalWrite(LED_A, HIGH);      
        digitalWrite(LED_B, HIGH); 
        digitalWrite(LED_C, HIGH);
        digitalWrite(LED_D, LOW);
        digitalWrite(LED_E, LOW);
        digitalWrite(LED_F, HIGH);
        digitalWrite(LED_G, HIGH); 
        break;
    }
}

~~~


## :robot: Link al proyecto
- [proyecto](https://www.tinkercad.com/things/2F3XAY52Hho-parcial-montacarga-sharlenne-azuaje-1d-/editel?sharecode=5Ais_VD4vX5r6o4amf1N4mMR5DwPREDrKTnEe2PA34w)
