# SMART-SHOPPING-CART
#include <SPI.h>
#include <MFRC522.h>
#include <LiquidCrystal.h>

// RFID and LCD pin definitions
#define SS_PIN 10  // Pin connected to SDA (RFID)
#define RST_PIN 9  // Pin connected to RST (RFID)

// Create instances for RFID and LCD
MFRC522 rfid(SS_PIN, RST_PIN);  // RFID instance
LiquidCrystal lcd(7, 2, 3, 4, 5, 6);  // LCD instance

float totalPrice = 0.0;  // Variable to store the total price

void setup() {
  // Initialize Serial communication, SPI, RFID, and LCD
  Serial.begin(9600);   // Initialize serial communication
  SPI.begin();          // Initialize SPI bus
  rfid.PCD_Init();      // Initialize RFID reader
  lcd.begin(16, 2);     // Initialize the LCD with 16 columns and 2 rows
  lcd.print("Scan an RFID");  // Initial message on LCD
  lcd.setCursor(0, 1);
  lcd.print("card...");

  Serial.println("Scan an RFID card...");
}

void loop() {
  // Look for new cards
  if (!rfid.PICC_IsNewCardPresent()) {
    return;  // No new card, continue to loop
  }

  // Select one of the cards
  if (!rfid.PICC_ReadCardSerial()) {
    return;  // Couldn't read card, continue to loop
  }

  // Print Card UID
  String cardUID = "";  // Variable to store the UID as a string
  for (byte i = 0; i < rfid.uid.size; i++) {
    if (rfid.uid.uidByte[i] < 0x10) {
      cardUID += "0";  // Add leading zero for single digits
    }
    cardUID += String(rfid.uid.uidByte[i], HEX);
  }

  // Clear LCD for new info
  lcd.clear();

  // Check card UID and print corresponding message
  if (cardUID.equalsIgnoreCase("73C51835")) {
    Serial.println("------------------------");
    Serial.println("SUGAR: TYPE: BROWNSUGAR ");
    Serial.println(" QTY: 1/2KG");
    Serial.println(" PRICE: 60.32 INR ONLY!!!");
    Serial.println("------------------------");
    totalPrice += 60.32;  // Add sugar price to the total
    
    // Display on LCD
    lcd.print("BROWNSUGAR 1/2KG");
    lcd.setCursor(0, 1);
    lcd.print("Price: 60.32 INR");

  } else if (cardUID.equalsIgnoreCase("6046C214")) {
    Serial.println("------------------------");
    Serial.println("RICE : TYPE: BASMATI  ");
    Serial.println(" QTY: 1KG");
    Serial.println(" PRICE: 232.53 INR ONLY!!!");
    Serial.println("------------------------");
    totalPrice += 232.53;  // Add rice price to the total

    // Display on LCD
    lcd.print("BASMATI RICE 1KG");
    lcd.setCursor(0, 1);
    lcd.print("Price: 232.53 INR");

  } else if (cardUID.equalsIgnoreCase("6318A70E")) {
    // Billing card
    Serial.println("BILLING...");
    delay(2000);
    Serial.print("Total Price: ");
    Serial.print(totalPrice, 2);  // Print the total price with 2 decimal places
    Serial.println(" INR");

    // Display total price on LCD
    lcd.print("Total: ");
    lcd.print(totalPrice, 2);
    lcd.setCursor(0, 1);
    lcd.print(" INR");

    totalPrice = 0.0;  // Reset the total price after billing

  } else {
    Serial.println("Unknown Card");
    lcd.print("Unknown Card");
  }

  // Halt PICC (card)
  rfid.PICC_HaltA();
  // Stop encryption on PCD (reader)
  rfid.PCD_StopCrypto1();

  delay(1000);  // Add a delay to prevent rapid reading
}
