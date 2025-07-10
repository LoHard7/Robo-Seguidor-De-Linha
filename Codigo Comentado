// ------------------------ Declaração dos pinos ------------------------
// Pinos dos motores (ponte H ou driver L298N, por exemplo)
const int IN1 = 27;   // Entrada 1 do motor A – controla sentido
const int IN2 = 14;   // Entrada 2 do motor A – controla sentido
const int IN3 = 12;   // Entrada 1 do motor B – controla sentido
const int IN4 = 13;   // Entrada 2 do motor B – controla sentido
const int ENA = 21;   // Enable (PWM) do motor A – controla velocidade
const int ENB = 22;   // Enable (PWM) do motor B – controla velocidade

// Pinos dos sensores infravermelhos de linha
const int IRMID = 33; // Sensor central (linha no meio do robô)
const int IRL1 = 25;  // Sensor interno esquerdo (próximo ao centro)
const int IRL2 = 26;  // Sensor externo esquerdo (mais afastado)
const int IRR1 = 32;  // Sensor interno direito
const int IRR2 = 35;  // Sensor externo direito

// ------------------------ Constantes de velocidade ------------------------
const int speed       = 200; // Velocidade base (0‑255) para andar reto
const int curveSpeed  = 150; // Velocidade reduzida para curvas suaves
const int searchSpeed = 150; // Velocidade usada na rotina de busca da linha
const int backSpeed   = 150; // Velocidade para a marcha‑ré na terceira tentativa

// ------------------------ Variáveis de estado ------------------------
String lastDirection = "forward"; // Guarda a última direção tomada
unsigned long lostTime = 0;       // Marca o instante em que a linha foi perdida
bool searching = false;           // Indica se o robô está em modo de busca

// ------------------------------ setup() ----------------------------------
void setup() {
  // Configura pinos dos motores como saída digital
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  pinMode(ENA, OUTPUT);
  pinMode(ENB, OUTPUT);

  // Configura pinos dos sensores como entrada digital
  pinMode(IRMID, INPUT);
  pinMode(IRL1, INPUT);
  pinMode(IRL2, INPUT);
  pinMode(IRR1, INPUT);
  pinMode(IRR2, INPUT);

  stopMotors(); // Garante que os motores iniciem parados
}

// ------------------------------ loop() -----------------------------------
void loop() {
  // Lê o estado de cada sensor infravermelho (HIGH = linha detectada)
  int MID = digitalRead(IRMID);
  int L1  = digitalRead(IRL1);
  int L2  = digitalRead(IRL2);
  int R1  = digitalRead(IRR1);
  int R2  = digitalRead(IRR2);

  // ----------- Tomada de decisão para pista de linha preta ---------------
  if (MID == HIGH) {              // Sensor central vê a linha → seguir reto
    forwardMotors();
    lastDirection = "forward";
    searching = false;
  }
  else if (L1 == HIGH) {          // Linha levemente à esquerda → corrija p/ direita
    adjustRight();
    lastDirection = "right";
    searching = false;
  }
  else if (R1 == HIGH) {          // Linha levemente à direita → corrija p/ esquerda
    adjustLeft();
    lastDirection = "left";
    searching = false;
  }
  else if (L2 == HIGH) {          // Linha bem à esquerda → curva acentuada p/ direita
    turnRight();
    lastDirection = "right";
    searching = false;
  }
  else if (R2 == HIGH) {          // Linha bem à direita → curva acentuada p/ esquerda
    turnLeft();
    lastDirection = "left";
    searching = false;
  }
  else {                          // Nenhum sensor vê a linha → linha perdida
    if (!searching) {             // Inicia contagem só na 1ª vez
      lostTime = millis();        // Guarda o instante da perda
      searching = true;           // Ativa modo de busca
    }
    searchLine();                 // Executa rotina de busca inteligente
  }

  delay(5); // Pequeno atraso para estabilidade do loop (5 ms)
}

// ========================= Funções de movimento ==========================
void forwardMotors() {            // Anda reto em velocidade base
  digitalWrite(IN1, LOW);         // Motor A gira para frente
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);         // Motor B gira para frente
  digitalWrite(IN4, HIGH);
  analogWrite(ENA, speed);        // PWM motor A
  analogWrite(ENB, speed);        // PWM motor B
}

void stopMotors() {               // Desliga ambos os motores
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
  analogWrite(ENA, 0);
  analogWrite(ENB, 0);
}

void adjustRight() {              // Correção suave p/ direita
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  analogWrite(ENA, speed);        // Roda esquerda (motor A) normal
  analogWrite(ENB, curveSpeed);   // Roda direita (motor B) mais lenta
}

void adjustLeft() {               // Correção suave p/ esquerda
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  analogWrite(ENA, curveSpeed);   // Roda esquerda mais lenta
  analogWrite(ENB, speed);        // Roda direita normal
}

void turnRight() {                // Curva acentuada p/ direita (pivot)
  digitalWrite(IN1, LOW);         // Motor A para frente
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH);        // Motor B para trás
  digitalWrite(IN4, LOW);
  analogWrite(ENA, speed);
  analogWrite(ENB, speed);
}

void turnLeft() {                 // Curva acentuada p/ esquerda (pivot)
  digitalWrite(IN1, HIGH);        // Motor A para trás
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);         // Motor B para frente
  digitalWrite(IN4, HIGH);
  analogWrite(ENA, speed);
  analogWrite(ENB, speed);
}

// ===================== Rotina de busca da linha ==========================
void searchLine() {
  unsigned long currentTime = millis(); // Tempo atual
  unsigned long elapsed = currentTime - lostTime; // Quanto tempo sem linha

  if (elapsed < 2000) {           // Até 2 s desde a perda
    // 1ª tentativa: gira para o lado da última curva feita
    if (lastDirection == "right") {
      digitalWrite(IN1, LOW);     // Motor A frente
      digitalWrite(IN2, HIGH);
      digitalWrite(IN3, HIGH);    // Motor B trás (gira p/ direita)
      digitalWrite(IN4, LOW);
    } else if (lastDirection == "left") {
      digitalWrite(IN1, HIGH);    // Motor A trás
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW);     // Motor B frente
      digitalWrite(IN4, HIGH);
    } else {                      // Se vinha reto, escolhe direita por padrão
      digitalWrite(IN1, LOW);
      digitalWrite(IN2, HIGH);
      digitalWrite(IN3, HIGH);
      digitalWrite(IN4, LOW);
    }
    analogWrite(ENA, searchSpeed);
    analogWrite(ENB, searchSpeed);
  }
  else if (elapsed < 4000) {      // Entre 2 s e 4 s sem linha
    // 2ª tentativa: inverte o sentido de rotação
    if (lastDirection == "right") {
      digitalWrite(IN1, HIGH);    // Gira p/ esquerda
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, HIGH);
    } else {
      digitalWrite(IN1, LOW);     // Gira p/ direita
      digitalWrite(IN2, HIGH);
      digitalWrite(IN3, HIGH);
      digitalWrite(IN4, LOW);
    }
    analogWrite(ENA, searchSpeed);
    analogWrite(ENB, searchSpeed);
  }
  else {                          // Mais de 4 s sem linha
    // 3ª tentativa: marcha‑ré, ambos motores para trás (giro amplo)
    digitalWrite(IN1, HIGH);      // Ambos motores para trás
    digitalWrite(IN2, LOW);
    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
    analogWrite(ENA, backSpeed);
    analogWrite(ENB, backSpeed);
  }
}
