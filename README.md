// # Weighting-machine-BUT3
// We built a weighting machine in a group project 



#include <Arduino.h>
#include <HX711.h>          // Bibliothèque pour le capteur de charge (cellule de pesée)
#include <WiFi.h>           // Bibliothèque pour la connexion Wi-Fi de l'ESP32
#include <esp_wpa2.h>       // Bibliothèque pour la connexion Wi-Fi sécurisée (Eduroam utilise WPA2-Enterprise)
#include <PubSubClient.h>   // Bibliothèque pour le protocole MQTT

float masse; 
float reading;

// Définition des broches de connexion du module HX711
const int LOADCELL_DOUT_PIN = 19;   // Broche de données du capteur
const int LOADCELL_SCK_PIN = 21;    // Broche d'horloge du capteur
const int Calibration_Weight = 50;  // Poids de calibration en grammes (connue)
HX711 scale;

// Identifiant du réseau Wi-Fi (Eduroam dans ce cas)
const char* ssid = "eduroam"; 

#define CONNECTION_TIMEOUT 10
int timeout_counter = 0;

// Paramètres du broker MQTT
//const char *mqtt_broker = "broker.hivemq.com";  // Exemple de broker public
const char *mqtt_broker = "172.23.136.18";       // Adresse IP du broker MQTT local
const char *topic = "IUT/balance";               // Nom du topic pour la publication
const char *mqtt_username = "";                  // Nom d’utilisateur MQTT (vide ici)
const char *mqtt_password = "";                  // Mot de passe MQTT (vide ici)
const int mqtt_port = 1883;                      // Port du broker MQTT (1883 = non sécurisé)
WiFiClient espClient;                            // Objet client Wi-Fi
PubSubClient client(espClient);                  // Objet client MQTT basé sur la connexion Wi-Fi

// Identifiants Eduroam pour l'authentification WPA2-Enterprise
#define EAP_IDENTITY "ewan.michellon@etu.univ-amu.fr"
#define EAP_PASSWORD "" // Mot de passe Eduroam (à compléter)
#define EAP_USERNAME "ewan.michellon@etu.univ-amu.fr"

// Fonction callback appelée à chaque réception d’un message MQTT
void callback(char *topic, byte *payload, unsigned int length) { 
  Serial.print("Le message a été envoyé sur le topic : "); 
  Serial.println(topic); 
  Serial.print("Message:"); 
  for (int i = 0; i < length; i++) { 
    Serial.print((char) payload[i]);  // Affiche le message reçu caractère par caractère
  } 
  Serial.println(); 
  Serial.println("-----------------------"); 
}

// Fonction d’initialisation
void setup() {
  Serial.begin(115200);  // Démarre la communication série
  scale.begin(LOADCELL_DOUT_PIN, LOADCELL_SCK_PIN); // Initialise le capteur HX711
  scale.set_scale();     // Définit l’échelle initiale (non calibrée)
  scale.tare();          // Remet la balance à zéro (tare)

  Serial.println("-----------------------------------------");
  Serial.println("Calibration");
  Serial.println("Put a known weight on the scale");

  // Connexion au réseau Wi-Fi Eduroam avec WPA2-Enterprise
  WiFi.disconnect(true);
  WiFi.begin(ssid, WPA2_AUTH_PEAP, EAP_IDENTITY, EAP_USERNAME, EAP_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {  // Attente de la connexion
    delay(500);
    Serial.print(F("."));
  }
  Serial.println("");
  Serial.println(F("L'ESP32 est connecté au WiFi !"));

  // Connexion au broker MQTT
  client.setServer(mqtt_broker, mqtt_port);  // Définition du serveur MQTT
  client.setCallback(callback);              // Association de la fonction callback

  // Tentative de connexion au broker MQTT
  while (!client.connected()) { 
    String client_id = "esp32-client-"; 
    client_id += String(WiFi.macAddress());  // Identifiant unique basé sur l’adresse MAC
    Serial.printf("La chaîne de mesure %s se connecte au broker MQTT", client_id.c_str()); 
 
    if (client.connect(client_id.c_str(), mqtt_username, mqtt_password)) { 
      Serial.println("La chaîne de mesure est connectée au broker."); 
    } else { 
      Serial.print("La chaîne de mesure n'a pas réussi à se connecter ... "); 
      Serial.print(client.state());          // Affiche le code d’erreur MQTT
    } 
  } 

  // Étape de calibration du capteur de pesée
  delay(5000);                               // Attente pour placer le poids de référence
  float x = scale.get_units(10);             // Mesure moyenne sur 10 lectures
  x = x / Calibration_Weight;                // Calcule le facteur d’échelle
  scale.set_scale(x);                        // Applique le facteur de calibration
  Serial.println("Calibration finished...");
  delay(1000);
}

// Boucle principale
void loop() {

  if (scale.is_ready()) {                    // Vérifie que le capteur HX711 est prêt
    reading = scale.get_units(10);           // Lit la valeur moyenne sur 10 mesures
    Serial.print("HX711 reading: ");
    Serial.println(reading);                 // Affiche la valeur brute de la pesée
  }

  masse = reading;                           // Stocke la valeur mesurée dans 'masse'
  client.publish(topic, String(masse).c_str()); // Envoie la masse au broker MQTT (convertie en texte)
  client.subscribe(topic);                   // S’abonne au topic (pour recevoir d’éventuels messages)
  client.loop();                             // Gère la communication MQTT (envoi/réception)
  delay(500);                                // Pause de 0,5 seconde entre chaque mesure
}
