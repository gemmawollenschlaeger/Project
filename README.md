boolean isGameOver = false;  // Game state
float mx;
float my;
float easing = 0.05;
int radius = 8;
int edge = 150;
int inner = edge + radius;
int xclouds = 0;
float xclouds1 = -400;
float xclouds2 = 400;
int yclouds, yclouds1, yclouds2 = 80;
int startTime; // Start time in milliseconds
int hunger = 100; // Starting hungerAdjustment value
int elapsedTime;
ArrayList<Food> foods;  // List of falling food objects
PFont font;             // For custom text rendering
int lastSpawnTime = 0;  // Time when the last food spawned
int spawnDelay = 2000;  // Delay between spawns (in milliseconds)
int hungerAdjustment;
int baseHunger;
int highScore;

// Setup for Game
void setup() {
  size(640, 640);
  noStroke();
  ellipseMode(RADIUS);
  rectMode(CORNERS);
  startTime = millis(); // Record the start time
  foods = new ArrayList<Food>();
}

void clouds() {
  noStroke();
  fill(245);
  ellipse(40, 40, 20, 20);
  ellipse(50, 60, 20, 20);
  ellipse(30, 60, 20, 20);
  ellipse(70, 56, 25, 25);
}

void draw() {
  noStroke();
  if (!isGameOver) {
    playGame();  // Main game logic
  } else {
    endScreen(); // End screen logic
  }

  // Create food at random intervals
  if (millis() - lastSpawnTime > spawnDelay) {
    spawnFoods(int(random(1, 5))); // Spawn 1-5 foods randomly
    lastSpawnTime = millis();
    spawnDelay = int(random(500, 3000)); // Random delay between 2 and 5 seconds
  }

  // Update and display all food
  for (int i = foods.size() - 1; i >= 0; i--) {
    Food f = foods.get(i);
    f.update();
    f.display();

    // Check if food reaches the bottom
    if (f.y > height) {
      if (f.isGood) {
        hungerAdjustment -= 5; // Penalize for unclicked good food
      }
      foods.remove(i); // Remove food
    }
  }
}

void playGame() {
  noStroke();
  background(#82F079);
  elapsedTime = millis() - startTime;

  fill(#79C7F0);
  rect(0, 0, 640, 370);

  if (abs(mouseX - mx) > 0.1) {
    mx = mx + (mouseX - mx) * easing;
  }
  if (abs(mouseY - my) > 0.1) {
    my = my + (mouseY - my) * easing;
  }

  mx = 320 + (mx - 320) / (320 / 90);
  my = 320 + (my - 320) / (320 / 90);

  // Body
  fill(#EF79F0);
  ellipse(640 / 2, 640 / 2 + 20, 80, 85);
  ellipse(640 / 2 - 40, 640 / 2 + 100, 20, 15);
  ellipse(640 / 2 + 40, 640 / 2 + 100, 20, 15);

  // Both Eyes Whites
  noStroke();
  if (hunger <= 60) {
    fill(128); // Gray color when condition is met
  } else {
    fill(0, 0, 0, 0); // Fully transparent
  }
  ellipse(640 / 2 + 40, 640 / 2 + 4, 20, 22);
  ellipse(640 / 2 - 40, 640 / 2 + 4, 20, 22);

  fill(245);
  stroke(0);
  ellipse(640 / 2 - 40, 640 / 2, 20, 20);
  ellipse(640 / 2 + 40, 640 / 2, 20, 20);

  // Eyeballs
  fill(35);
  ellipse(mx - 40, my, radius, radius);
  ellipse(mx + 40, my, radius, radius);

  // Clouds
  pushMatrix();
  translate(xclouds, yclouds);
  xclouds++;
  clouds();
  popMatrix();

  pushMatrix();
  translate(xclouds1, yclouds1);
  xclouds1 = xclouds1 + 0.75;
  clouds();
  popMatrix();

  pushMatrix();
  translate(xclouds2, yclouds2);
  xclouds2 = xclouds2 + 1.1;
  clouds();
  popMatrix();

  if (xclouds > 680) {
    xclouds = -80;
    yclouds = int(random(-20, 140));
  }

  if (xclouds1 > 680) {
    xclouds1 = -80;
    yclouds1 = int(random(-20, 140));
  }

  if (xclouds2 > 680) {
    xclouds2 = -80;
    yclouds2 = int(random(-20, 140));
  }

  // Hunger bar update
  baseHunger = 100 - int(100 * (elapsedTime / 30000.0));
  hunger = baseHunger + hungerAdjustment;
  hunger = constrain(hunger, 0, 100); 

  // hunger bar display
  int barWidth = 300; 
  int barHeight = 50; 
  float hungerRatio = hunger / 100.0;
  int currentWidth = int(hungerRatio * barWidth);
  color barColor = lerpColor(color(0, 255, 0), color(255, 0, 0), 1 - hungerRatio);

  // Position of the bar
  int barX = width - barWidth - 20;
  int barY = height - barHeight - 20;
  rectMode(CORNER);
  fill(barColor);
  noStroke();
  rect(barX, barY, currentWidth, barHeight);

  // Outline of bar
  noFill();
  stroke(0);
  rect(barX, barY, barWidth, barHeight);

  // Percentage of Hunger 
  fill(0);
  textSize(20);
  textAlign(CENTER, CENTER);
  text(hunger + "%", barX + barWidth / 2, barY + barHeight / 2);

  // Check game over condition
  if (hunger <= 0) {
    isGameOver = true;
    highScore = elapsedTime/1000;
  }
}

void endScreen() {
  background(50);  
  fill(255);  
  textAlign(CENTER, CENTER);
  text("Game Over", width / 2, height / 2 - 40);
  text("Andrey Rosey Died", width / 2, height / 2 - 20);
  text("Press 'R' To Bring Them Back To Life", width / 2, height / 2 + 10);
  text("Your Score:", width / 2, height / 2 + 40);
  text(highScore, width / 2, height / 2 + 70);
  noLoop(); // Stop the game loop
}

void keyPressed() {
  if (isGameOver && (key == 'r' || key == 'R')) {
    restartGame();
  }
}

void restartGame() {
  hungerAdjustment = 0;  // Reset hunger adjustment
  isGameOver = false;  // Reset game state
  startTime = millis();  // Reset start time to current time
  elapsedTime = 0;  // Reset elapsed time
  foods.clear();  // Clear the food array
  loop();  // Restart the game loop
}

void mousePressed() {
  boolean clickedFood = false;

  // Check if any food is clicked
  for (int i = foods.size() - 1; i >= 0; i--) {
    Food f = foods.get(i);
    if (f.isClicked(mouseX, mouseY)) {
      clickedFood = true;
      if (f.isGood) {
        hungerAdjustment += 10; // Good food adds health
      } else {
        hungerAdjustment -= 10; // Bad food reduces health
      }
      foods.remove(i); // Remove the clicked food
      break;
    }
  }

  // If no food is clicked, reduce health
  if (!clickedFood) {
    hungerAdjustment -= 5;
  }
}

void spawnFoods(int count) {
  for (int i = 0; i < count; i++) {
    foods.add(new Food(random(width), random(-50, -20), random(1) > 0.5));
  }
}

class Food {
  float x, y;       // Position
  float speed;      // Falling speed
  boolean isGood;   // Whether the food is good or bad

  Food(float x, float y, boolean isGood) {
    this.x = x;
    this.y = y;
    this.isGood = isGood;
    this.speed = random(4,8); // Slower, random speed
  }
  void update() {
    y += speed; // Move the food down
  }

  void display() {
    noStroke();
    if (isGood) {
      fill(0, 255, 0); // Green for good food
    } else {
      fill(255, 0, 0); // Red for bad food
    }
    ellipse(x, y, 20, 20); // Draw food
  }

  boolean isClicked(float mx, float my) {
    return dist(mx, my, x, y) < 24; // Check if mouse click is within the food
  }
}
