# Processadors_Digitals_Practica_2
# Practica 2

Aquesta pràctica consisteix en familiaritzar-nos amb el concepte d’interrupcions i es pot aplicar als programes que escrivim en properes pràctiques.

# Interrupcions

Les interrupcions ens permeten detectar events externs al programa i tractar-los de foma eficient. Per posar un exemple, en comptes de revisar cada x segons l’estat d’un pin, podem rebre una mena d’alerta quan l’estat del pin canviï i per tant, podem pausar el procés que estiguessim fent, tractar el canvi d’estat del pin i continuar amb el procés que estavem fent.
Aixo ens permet dur a terme processos mentres esperem un event extern, cosa que resulta en una resposta quasi immediata a events exteriors com poden ser la poslació d’un botó, un canvi de voltatge en un pin, etc.

Existeixen diversos tipus d’interrupcions:
- Event de hardware (les que utilitzem a la pràctica A)
- Event programat o *Timer* (les que utilitzem a la pràctica B)
- Crides per software

#### Interrupcions per event de hardware
Aquest tipus d'interrupcions ocorren; com indica el nom, per events externs a la placa. Per exemple, la tecla d'un teclat, un clic d'un ratolí o; com fem a la pràctica A, la polsació d'un botó.
Aquestes interrupcions s'han dinicialitzar i desactivar amb les seve funcions pròpies que veurem més endavant.

#### Interrupcions per events programats
Aquestes interrupcions són les que vam utilitzar a la **Pràctica_1.2.cpp** per comptes de la funció `delay()`. Aquest tipus d'interrupcions utilitzen el rellotge intern de la placa i ens permeten detectar un cert "temps" amb el qual podem treballar.

flowchart TD
    A[Programa] -->B[Evento]
    B --> D[Loop]
    D --> E[Programa Principal]
    B -->|Hardware Interruption| C[Interrupción]
    C --> B

## Pràctica A

Aquesta primera pràctica consisteix a detectar i tractar la polsació d’un botó connectat a la placa **********ESP32********** introduïnt una interrupció de tipus ********GPIO********. Aquesta interrupció detectarà el flanc de baixada d’un botó (quan es deixa de polsar) durant 60 segons. Per fer-ho inicialitzem el botó al codi per a que la placa el pugui detectar i el pin on l’hem connectat.

```sql
struct Button {
  const uint8_t PIN;
  uint32_t numberKeyPresses;
  bool pressed;
};

Button button1 = {18, 0, false};
```

Les interrupcions per **GPIO** cal activar-les i desactivar-les al programa. Aixo ho fem amb les funcions `attachInterrupt(button1.PIN, isr, FALLING);` i `detachInterrupt(button1.PIN);` Dins d'aquestes funcions hem d'indicar el pin que genera la interrupció, i el *mode* o el moment en que es passarà de codi principal a tractar la interrupció.
### Funció `setup()`
A la funció `setup()` activarem la interrupció amb la funció `attachInterrupt()`que hem vist avans.
```
void setup() {
  Serial.begin(115200);
  pinMode(button1.PIN, INPUT_PULLUP);
  attachInterrupt(button1.PIN, isr, FALLING);
}
```

### Funció `loop()`
En aquesta funció inclourem el tractament de l'event i el comptador per parar el programa als 60 segons, i desactivarem la interrupció.
```
void loop() {
  if (button1.pressed) {
    Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
    button1.pressed = false;
  }
  //Detach Interrupt after 1 Minute
  static uint32_t lastMillis = 0;
  if (millis() - lastMillis > 60000) {
    lastMillis = millis();
    detachInterrupt(button1.PIN);
    Serial.println("Interrupt Detached!");
  }
}
```


## Pràctica B
En aquesta pràctica sen's demana generar i monitoritzar una interrupció amb un *timer*. Aquest tipus d'interrupcions utilitzen el rellotge intern de la placa. En aquest cas utilitzarem aquest rellotge per crear les interrupcions cada 80 ms.
 
### Funció `setup()`
En aquesta funció inicialitzarem aquesta interrupció per *timer* 
```
void setup() {
    Serial.begin(115200);
    timer = timerBegin(0, 80, true);
    timerAttachInterrupt(timer, &onTimer, true);
    timerAlarmWrite(timer, 1000000, true);
    timerAlarmEnable(timer);
}
```
Aqui també indiquem el temps que volem entre una interrupció i una altra; en aquest cas 80 ms.

### Funció `loop()`
En aquesta funció monitoritzem quants cops s'inicia la interrupció i ho mostrem per pantalla.
```
void loop() {
    if (interruptCounter > 0) {
        portENTER_CRITICAL(&timerMux);
        interruptCounter--;
        portEXIT_CRITICAL(&timerMux);
        totalInterruptCounter++;
        Serial.print("An interrupt as occurred. Total number: ");
        Serial.println(totalInterruptCounter);
    }
}
```
___
