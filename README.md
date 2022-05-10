## Invio dati ad un server remoto

Si include la libreria che gestisce l'interfaccia radio WIFI a 2.4GHz del modulo ESP32:

    #include <WiFi.h>
Si include la libreria che permette la gestione di una connessione client ad un server HTTP:

    #include <HTTPClient.h>

Si definisce il numero massimo di tentativi di connessione al server

    #define RETRY_LIMIT  20

Si inseriscono l'SSID del router al quale il nostro ESP32 deve connettersi e la sua password di accesso: 

    const char* SSID ="..............";
    const char* PASSWORD = ".............";
    
Creo l'oggetto client HTTP con cui gestire la connessione al server web remoto:

     HTTPClient http;

### Fase di Setup

   
    WiFi.begin(SSID,PASSWORD);
 
Attendiamo fino a quando l'intefaccia WIFI non commuta nello stato WL_CONNECTED di connessione avvenuta: 
    
    while (WiFi.status()!= WL_CONNECTED){
      delay(500);
      Serial.print(".");
    }
    
Stampo informazioni di connessione avenuta, l'indirizzo IP dell'ESP32 assegnato dal router:
    
    Serial.println("");
    Serial.println("WiFi connected");
    Serial.println("IP Address");
    Serial.println(WiFi.localIP()); 
 
### Fase di loop

Per il momento simuliamo i dati che dovrebbero acquisire i tre sensori di pressione, temperatura, umidità, carica residua della batteria di alimentazione


    float humidity = random(10,90);
    float temperature = random(10,30);
    float pressure = random(950,1100);
    
In realtà in ogni ciclo di misura dovrei acquisire l'informazione dai sensori di monitoraggio collegati all'esp32. Si stabilisce una connessione con il server:
    
 
   

      http.begin("http://192.168.71.174/esp_get.php");

      http.addHeader("Content-Type", "application/x-www-form-urlencoded");

 Si inseriscono i dati che si vogliono inviare secondo la notazione nome=valore separati dall'ampersand &:
  
      int httpResponseCode = http.POST("&temp="+String(temp)+"&humidity="+String(humidity)+"&probe="+String(pressure)+"&charge=5&device=1");
      
Si controlla poi il codice di risposta del server che dovrebbe essere 200 (ok):

      if (httpResponseCode >0){
          //check for a return code - This is more for debugging.
        String response = http.getString();
        Serial.println(httpResponseCode);
        Serial.println(response);
      }
      else{
        Serial.print("Error on sending post");
        Serial.println(httpResponseCode);
      }
      
  Si chiude la transazione:
  
      http.end(); 
    
 Aspetto un minuto tra un invio e il successivo:
 
    delay(60000);

