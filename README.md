# esp32

```
// Template ID, Device Name and Auth Token are provided by the Blynk.Cloud
// See the Device Info tab, or Template settings
#define BLYNK_TEMPLATE_ID ""
#define BLYNK_DEVICE_NAME ""
#define BLYNK_AUTH_TOKEN ""


// Comment this out to disable prints and save space
#define BLYNK_PRINT Serial


#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>

char auth[] = BLYNK_AUTH_TOKEN;

// Your WiFi credentials.
// Set password to "" for open networks.
char ssid[] = "";
char pass[] = "";

BlynkTimer timer; 

int check_water_lvl_ID; //https://community.blynk.cc/t/using-blynktimer-or-simpletimer/53326

//long time_to_wait; // 3 seconds // test -- Using variables instead of hard-coded numbers

// DOBRA PROJEKT NA JAKICH PINACH TO ROBIE 
//relay jest na 16 czyli 8pin z prawej
//sensor jest 39 czyli 16pin z lewej
//i tak WAZNE dajesz zasilanie 3,3V bo inaczej nie  pojdzie ito jest 19pin z lewej

// ktore piny 
//https://randomnerdtutorials.com/esp32-pinout-reference-gpios/

#define relay1_pin 16 // pompa // na plytce to 5 pin z lewej
#define sensor 39 // czujnik na analogu // 11 pin z lewej //define analog input pin number connected to moisture sensor

const int dry = 3450; //warosc kiedy jest suchy - stan na procent
const int mid = 3000; // wartosc na warunek // dostosuj go potem do innej wartosci
const int wet = 1400; //wartosc kiedy mokry -- stan na procent

// zamiast state to mode i nr trybu

bool mode1_button = 0;
bool mode2_button = 0;
bool mode3_button = 0;

long slider_delay = 0; // to do slidera ile czasu do fromuly


int sensorData = 0; // opisz w inz ze mozna i na procenty  
int output = 0; // stan do procentow ale narazie nie uzyte

int pinValue = 0; // stan do v4 czyli mienika
int sensorVal = 0; // stan na analog do esp na czujnik


// https://community.blynk.cc/t/using-blynktimer-or-simpletimer/53326 important to timer setup

//Change the virtual pins according the rooms
#define button1_vpin    V1 // 1st mode on/off
#define button2_vpin    V2 // auto 
#define button3_vpin    V3 // slider
#define button4_vpin    V4 // led do trybu 1
#define button5_vpin    V5 // miernik czujnika wilgotnosci
#define button6_vpin    V6 // on off czujnika
//------------------------------------------------------------------------------
// This function is called every time the device is connected to the Blynk.Cloud
// Request the latest state from the server
BLYNK_CONNECTED() {
  Blynk.syncVirtual(button1_vpin);
  Blynk.syncVirtual(button2_vpin);
  Blynk.syncVirtual(button3_vpin);
  Blynk.syncVirtual(button4_vpin); 
  Blynk.syncVirtual(button5_vpin);
  Blynk.syncVirtual(button6_vpin);
}
//--------------------------------------------------------------------------
// This function is called every time the Virtual Pin state change
//i.e when web push switch from Blynk App or Web Dashboard

/*## inz
//  nie kompiluje sie przez to nie dodawaj tego -- to tez napisz do inz ze myslales ze trzeba xd
BLYNK_CONNECTED() {
    Blynk.syncAll();
}
*/

/*
BLYNK_WRITE(button1_vpin) // https://www.youtube.com/watch?v=UBQCaxfeBKY // chlep
  { 
     
    mode1_button = param.asInt(); // bool
    
//   Blynk.notify("You just watered your plant."); // jak chcesz miec info
if(mode1_button == HIGH) {// jesli przycisk on/off jest wcisniety
  digitalWrite(relay1_pin, mode1_button); // wlaczam pompke na 2sek
  Blynk.virtualWrite(button4_vpin, HIGH); // zapalam led - tez
  Serial.println("tryb 1 wlaczony");
  delay(2000); 
  //------- koncze i wygaszam tryb
  mode1_button = LOW;
  digitalWrite(relay1_pin, mode1_button); // wylaczam pompe
  Blynk.virtualWrite(button1_vpin, LOW); // wylacza przycisk do stanu wyjsciowego czyli OFF
  Blynk.virtualWrite(button4_vpin, LOW); // wygaszam led 
  Serial.println("tryb 1 wylaczony");
  delay(500);
  
  }
  else{ 
  mode1_button = LOW;
  digitalWrite(relay1_pin, mode1_button);
  Serial.println("tryb 1 wylaczony");
  delay(100); 
    }
 // }
  }
*/
// orginal trybu 1 ---- zostaw to sobie byc w inz mogl pisac ze tak myslales zle na poczatku itp 

//--------------------------------------------------------------------------
//############## code for mode 1 ###########################################

BLYNK_WRITE(button1_vpin) // https://www.youtube.com/watch?v=UBQCaxfeBKY // chlep
  { 
    
    mode1_button = param.asInt(); // bool
    
    if(mode1_button == 1) {// jesli przycisk on/off jest wcisniety
        modeone(); // wywolanie funkcji wlaczajacej tryb 1 i led 
    }

    if(mode1_button == 0)
    {
  digitalWrite(relay1_pin, mode1_button); // wylaczam pompe . relay1_state or LOW
  Blynk.virtualWrite(button4_vpin, LOW); // wygaszam led 
  Serial.println("tryb 1 wylaczony");
    }
    
  }
//--------------------------------------------------------------------------
void modeone() // pelta co 100mils na tryb 1
{
 
        digitalWrite(relay1_pin, mode1_button); // wlaczam pompke 
        Blynk.virtualWrite(button4_vpin, HIGH); // zapalam led - tez
        Serial.println("tryb 1 wlaczony");
        
}
//############## code for mode 1 ###########################################
//--------------------------------------------------------------------------
//############## code for mode 2 ###########################################
BLYNK_WRITE(button2_vpin) {  

mode2_button = param.asInt(); // bool zmien 

if(mode2_button == HIGH) {
  Serial.println(" wlaczono tryb auto - ustaw czas podlania ");
}

if(mode2_button == LOW){
  Serial.println(" wylaczono tryb auto ");
  digitalWrite(relay1_pin, mode2_button); // pump off
 }
}
//--------------------------------------------------------------------------

BLYNK_WRITE(button3_vpin)
{ // https://www.youtube.com/watch?v=vp2Cd_2yWaI - kod na slider

  if(mode2_button == HIGH) {// tryb auto jest wsciniety, wcisniety to high
 
  pinValue = param.asInt(); // assigning incoming value from pin V3 to a variable
 
  Serial.print("lutriush zostanie podlany za: ");
  Serial.print(pinValue);
  Serial.println(" dni ");
  slider();
  
    }
    else{
       Blynk.virtualWrite(button3_vpin, LOW); // wylacza slider do pozycji poczatkowej ZERO heh
    }
}
//--------------------------------------------------------------------------
void slider()
{
    slider_delay = (pinValue*1000);
  
  //Serial.println(czas);
  delay(slider_delay);
  digitalWrite(relay1_pin, HIGH); //water pump on
  delay(2000);                                                                                  // policz ile czasu woda bd sie lapa uwzgledniajac dlugosc rurki
  digitalWrite(relay1_pin, LOW); //water pump off
  Blynk.virtualWrite(button2_vpin, LOW); // wylacza przycisk do stanu wyjsciowego czyli OFF
  Blynk.virtualWrite(button3_vpin, LOW); // wylacza slider do pozycji poczatkowej ZERO heh
}


// pierwsze podejscie - zostawiam sobie pod inzynierska
/*
 * BLYNK_WRITE(button2_vpin) {  
mode2_button = param.asInt(); // bool zmien 
if(mode2_button == HIGH) {
  Serial.println(" wlaczono tryb auto - ustaw czas podlania ");
  // call another void
}
if(mode2_button == HIGH){
  Serial.println(" wylaczono tryb auto ");
  digitalWrite(relay1_pin, mode2_button);
 }
}
//--------------------------------------------------------------------------
BLYNK_WRITE(button3_vpin)
{ // https://www.youtube.com/watch?v=vp2Cd_2yWaI - kod na slider
   pinValue = param.asInt(); // assigning incoming value from pin V3 to a variable
  
  if(mode2_button == HIGH) {// tryb auto jest wsciniety, wcisniety to high
 
slider();
 
  Serial.print("lutriush zostanie podlany za: ");
  Serial.print(pinValue);
  Serial.println(" dni ");
  
  czas = (pinValue*1000);
  Serial.println(czas);
  delay(czas);
  Serial.println(" elo");
  digitalWrite(relay1_pin, HIGH); //pinValue
  delay(2000); // czas podlewania uzaleznisz od dlugosci rurki
  digitalWrite(relay1_pin, LOW);
  
  Blynk.virtualWrite(button2_vpin, LOW); // wylacza przycisk do stanu wyjsciowego czyli OFF
  Blynk.virtualWrite(button3_vpin, LOW); // wylacza slider do pozycji poczatkowej ZERO heh
    }
}
*/

//############## code for mode 2 ###########################################
//--------------------------------------------------------------------------
//#### code for mode 3##################################################### code for mode 3

BLYNK_WRITE(button6_vpin) {   // on of soil moisture // https://docs.blynk.io/en/blynk.edgent/api/blynk-timer
  
  mode3_button = param.asInt(); // bool
                                                                // to chyba dziala ale nie jest sprawdzane po nacisnieciu przycisku 3 wiec ndadal moge nacisnac inne przyciski 
        Blynk.virtualWrite(button1_vpin,LOW); // Turn Button 1 off    // jezeli chcialbym by wtrakcie pracy nie dalo sie nacisnac innych trybow to mysialbym na to napisac voida
        Blynk.virtualWrite(button2_vpin,LOW); // Turn Button 2 off // tak dziala i jest spo ko ale moze dzialac lepiej 
        // Update the other button variables to avoid confusion...
        mode1_button = 0; 
        mode2_button = 0;


if(mode3_button == 1)  // https://community.blynk.cc/t/using-blynktimer-or-simpletimer/53326
      {
        timer.enable(check_water_lvl_ID);// wlaczam interwal spr poziom wilgotnosci 
        Serial.println("wlaczono tryb 3");
        Serial.println("warunek spr czujnika");        
      }

  if(mode3_button != 1){
    Serial.println(" wylaczono tryb czujnika gleby ");   
    digitalWrite(relay1_pin, LOW);
     timer.disable(check_water_lvl_ID); // wylaczam interwal spr poziom wilgotnosci 
   
    }
  }
//--------------------------------------------------------------------------
void check_water_lvl() // podlewa kwaitek jak speni warunek na okolo 3 sek
{
           if(sensorVal >= mid)
       {        
             sendSensor();// wywolanie sensor data // pompka on
       }
     else{
            sendSensor2(); // wywolanie sensor data2 // pompka off
      }
  }
//--------------------------------------------------------------------------

void sendSensor() // gdy pompka jest wlaczona
{
  sensorVal = analogRead(sensor); //reading the sensor on 39

  if ( isnan(sensorVal) ){
     Serial.println("Failed to read from Hygrometer Soil Moisture sensor!"); // security
    return;
  } else {
    Serial.print("sensor lvl value - ");
     Serial.println(sensorVal);
    // When the plant is watered well the sensor will read a value 380~400, I will keep the 400 
    // value but if you want you can change it below. 
          // sensorVal = constrain(sensorVal,1400,3450);  //Keep the ranges!
          // output = map(sensorVal,1400,3450,100,0);  //Map value : 1400 will be 100 and 3450 will be 0
       // Serial.println("procentage of sensor - ");
       // Serial.print(sensorVal);
       // Serial.println("%");
       
        digitalWrite(relay1_pin, HIGH);
  }
}
//--------------------------------------------------------------------------

void sendSensor2(){ // gdy pompka ma byc wylaczona

  sensorVal = analogRead(sensor); //reading the sensor on 39

  if ( isnan(sensorVal) ){
     Serial.println("Failed to read from Hygrometer Soil Moisture sensor!"); // security
    return;
  } else {
    Serial.print("sensor lvl value - ");
     Serial.println(sensorVal);
       
        digitalWrite(relay1_pin, LOW);
        switch6off();
  }
}
//--------------------------------------------------------------------------

void switch6off(){ 
  if(sensorVal >= mid){
     Blynk.virtualWrite(button6_vpin, LOW); // przycisk wylacza sie
  }
  else{
     Blynk.virtualWrite(button6_vpin, HIGH);  // przycisk wlacza sie
  }
}
//--------------------------------------------------------------------------

void myTimerEvent() // wyswietla dane z czujnika na miernik co 1.25 sek
{
  sensorVal = analogRead(sensor); // no zeby miernik wiedzial w czym ma byc zakres czyli od 0 do 4096
  
 // output = analogRead(sensor); //tu jest zakres od 0 do 100 procent 
  //sensorVal = constrain(sensorVal,1400,3450);  //Keep the ranges!
  //output = map(sensorVal,1400,3450,100,0);  //Map value : 1400 will be 100 and 3450 will be 0
        
  // You can send any value at any time.
  // Please don't send more that 10 values per second.
  //Blynk.virtualWrite(button5_vpin, millis() / 1000);
   
  Blynk.virtualWrite(button5_vpin, sensorVal); // pokazuje na mierniku wartosc od 1400 do 3450 
 
  //Blynk.virtualWrite(button5_vpin, output); // pokazuje na mierniku w procentach

} // to co zle do inzynierskiej ze bledy
 /*
  if(relay3_state == 1) { // przycisk tryb 3 jest wlaczony
  Serial.println(" wlaczono tryb czujnika gleby ");
  //delay(500); // zeby w konsoli nie bylo za szybko
  //------------------------------------------------------------ zaczecie czego kolwiek w tej funkcji
// to olej chyba i dodaj ten warunek do petli
    if (sensorVal >= mid){
      // sensorVal = param.asInt();
     // soil_loop();// wywolanie funkcji -- sprawdz warunek
      sendSensor();
        }
    
    else{
        digitalWrite(relay1_pin, LOW); // wylaczasz pompe 
  // info w konsoli
  Serial.print("lutriush zostal podlany i ma luz ");
  Serial.println(sensorVal);
  //delay(1500);
  Blynk.virtualWrite(button6_vpin, LOW);
    }
      }
 
   //------------------------------------------------------------- koniec 
    if(relay3_state == 0){
    Serial.println(" wylaczono tryb czujnika gleby ");
  }
}
*/
//#### code for mode 3##################################################### code for mode 3
//--------------------------------------------------------------------------

// to odpuszczam chyba bo nie musze tego pisac
// -------------------------------------------------- //  z czujnikiem wilgotonosci v5
//BLYNK_WRITE(button5_vpin) {  // part 1 https://www.youtube.com/watch?v=IvzBPXqyUm4  part 2 https://www.youtube.com/watch?v=xyM2RECZBTo
 // test na potencjometrze https://randomnerdtutorials.com/esp32-adc-analog-read-arduino-ide/
//--------------------------------------------------------------------------
void setup()
{
  // Debug console
  Serial.begin(115200);
  //--------------------------------------------------------------------
 //  pinMode(button1_pin, INPUT_PULLUP);  nieuzywam jednak
  //--------------------------------------------------------------------
  pinMode(relay1_pin, OUTPUT);
  pinMode(sensor, OUTPUT);
  //--------------------------------------------------------------------
  //During Starting all Relays should TURN OFF
  digitalWrite(relay1_pin, HIGH);
  digitalWrite(sensor, HIGH);
  //--------------------------------------------------------------------
  //https://github.com/blynkkk/blynk-library/blob/master/examples/GettingStarted/PushData/PushData.ino#L30
  // Setup a function to be called every second
  timer.setTimeout(3600000L, [] () {} ); // dummy/sacrificial Function // https://community.blynk.cc/t/using-blynktimer-or-simpletimer/53326
  timer.setInterval(1250L, myTimerEvent);
  check_water_lvl_ID = timer.setInterval(2250L, check_water_lvl); //Disabling/Enabling a timer to achieve the same result  https://community.blynk.cc/t/using-blynktimer-or-simpletimer/53326
   //--------------------------------------------------------------------
  Blynk.begin(auth, ssid, pass);
  // You can also specify server:
  //Blynk.begin(auth, ssid, pass, "blynk.cloud", 80);
  //Blynk.begin(auth, ssid, pass, IPAddress(192,168,1,100), 8080);
  //--------------------------------------------------------------------
  //update button states || wysyla wartosc z kodu arduino do widzetu w blynk
  Blynk.virtualWrite(button1_vpin, mode1_button);
  Blynk.virtualWrite(button2_vpin, mode2_button);
  Blynk.virtualWrite(button3_vpin, mode2_button);
  Blynk.virtualWrite(button4_vpin, pinValue);
  Blynk.virtualWrite(button5_vpin, mode1_button);
  //--------------------------------------------------------------------
}// https://docs.blynk.io/en/legacy-platform/legacy-articles/how-to-display-any-sensor-data-in-blynk-app info do inz
// esp32 pinout https://www.instructables.com/ESP32-Internal-Details-and-Pinout/

void loop()
{
  Blynk.run(); // funkcje wbudowane z bibl z blynk
  timer.run(); // https://community.blynk.cc/t/using-blynktimer-or-simpletimer/53326
  // You can inject your own code or combine it with other sketches.
  // Check other examples on how to communicate with Blynk. Remember
  // to avoid delay() function!
  // keep loop clean  https://docs.blynk.io/en/legacy-platform/legacy-articles/keep-your-void-loop-clean --- info do opisow inzynierskiej co i jak
  }
  ```
