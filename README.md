# Guess-number-game

#include <SFML/Graphics.hpp>
#include <iostream>
#include <cstdlib>
#include <ctime>
#include <sstream>
#include <vector>

int main() {
    // Initialize random seed
    std::srand(static_cast<unsigned int>(std::time(nullptr)));
    int secretNumber = std::rand() % 100 + 1;
    int attemptsLeft = 10;
    int userGuess = 0;
    std::string inputString;
    bool gameWon = false;

    // Visual effect variables
    int flashFrames = 0;
    sf::Color bgColor = sf::Color::Black;

    // Create window
    sf::RenderWindow window(sf::VideoMode(400, 300), "Guess the Number Game");
    sf::Font font;

    // Try multiple font paths
    std::vector<std::string> fontPaths = {
        "/usr/share/fonts/truetype/liberation/LiberationSans-Regular.ttf",
        "/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf",
        "/usr/share/fonts/truetype/freefont/FreeSans.ttf"
    };

    bool fontLoaded = false;
    for (const auto& path : fontPaths) {
        if (font.loadFromFile(path)) {
            fontLoaded = true;
            std::cout << "Loaded font: " << path << std::endl;
            break;
        }
    }

    if (!fontLoaded) {
        std::cout << "Warning: Could not load any system font. Text will not display." << std::endl;
    }

    // Set up text objects with better positioning
    sf::Text titleText("Guess the Number (1-100)", font, 24);
    titleText.setPosition(20.f, 20.f);
    titleText.setFillColor(sf::Color::White);

    // Create a separate input area with clear separation
    sf::Text promptText("Your guess:", font, 20);
    promptText.setPosition(20.f, 80.f);  // Moved down to make space
    promptText.setFillColor(sf::Color::White);

    // Create a visible input box
    sf::RectangleShape inputBox(sf::Vector2f(80.f, 30.f));
    inputBox.setPosition(120.f, 80.f);  // Positioned after the prompt
    inputBox.setFillColor(sf::Color(30, 30, 30));  // Dark gray background
    inputBox.setOutlineThickness(2.f);
    inputBox.setOutlineColor(sf::Color::White);

    // Position input text inside the box
    sf::Text inputDisplay("", font, 20);
    inputDisplay.setPosition(125.f, 82.f);  // Inside the input box
    inputDisplay.setFillColor(sf::Color::Green);

    sf::Text feedbackText("You have 10 attempts.", font, 20);
    feedbackText.setPosition(20.f, 130.f);  // Moved down
    feedbackText.setFillColor(sf::Color::Yellow);

    sf::Text attemptsText("Attempts left: " + std::to_string(attemptsLeft), font, 18);
    attemptsText.setPosition(20.f, 170.f);  // Moved down
    attemptsText.setFillColor(sf::Color::Cyan);

    // Main game loop
    while (window.isOpen()) {
        sf::Event event;

        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed)
                window.close();

            if (event.type == sf::Event::TextEntered) {
                if (event.text.unicode == '\b' && !inputString.empty()) {
                    inputString.pop_back();
                } else if (event.text.unicode >= '0' && event.text.unicode <= '9') {
                    if (inputString.size() < 3) {
                        inputString += static_cast<char>(event.text.unicode);
                    }
                } else if (event.text.unicode == '\r' && !inputString.empty() && !gameWon && attemptsLeft > 0) {
                    userGuess = std::stoi(inputString);
                    attemptsLeft--;

                    if (userGuess == secretNumber) {
                        gameWon = true;
                        feedbackText.setString("Congratulations! Correct!");
                        feedbackText.setFillColor(sf::Color::Green);
                        flashFrames = 60;
                    } else if (userGuess < secretNumber) {
                        feedbackText.setString("Too small!");
                        feedbackText.setFillColor(sf::Color::Yellow);
                    } else {
                        feedbackText.setString("Too big!");
                        feedbackText.setFillColor(sf::Color::Yellow);
                    }

                    if (attemptsLeft <= 0 && !gameWon) {
                        feedbackText.setString("Game over! Number: " + std::to_string(secretNumber));
                        feedbackText.setFillColor(sf::Color::Red);
                    }
                    inputString.clear();
                    attemptsText.setString("Attempts left: " + std::to_string(attemptsLeft));
                }
            }
        }

        // Update visual effects
        if (flashFrames > 0) {
            if (flashFrames % 10 < 5) {
                bgColor = sf::Color::Green;
            } else {
                bgColor = sf::Color::Black;
            }
            flashFrames--;
        } else if (gameWon) {
            bgColor = sf::Color(0, 100, 0);
        } else {
            bgColor = sf::Color::Black;
        }

        inputDisplay.setString(inputString);

        // Render
        window.clear(bgColor);
        if (fontLoaded) {
            window.draw(titleText);
            window.draw(promptText);
            window.draw(inputBox);
            window.draw(feedbackText);
            window.draw(attemptsText);
            window.draw(inputDisplay);
        }
        window.display();
    }

    return 0;
}
