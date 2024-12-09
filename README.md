# ErikCH
#include <iostream>
#include <vector>
#include <algorithm>
#include <random>
#include <ctime>
using namespace std;

// Card struct to hold value and suit
struct Card {
    string suit;
    string rank;
    int value;
};

// Functions for deck handling
vector<Card> createDeck();
void shuffleDeck(vector<Card>& deck);
Card drawCard(vector<Card>& deck);

// Hand management
int calculateHandValue(const vector<Card>& hand, bool& isSoft);
void displayHand(const vector<Card>& hand, const string& name);

// Probability calculations
double calculateBustProbability(const vector<Card>& deck, int currentTotal);
double calculateWinningProbability(const vector<Card>& deck, int playerTotal, int dealerVisible);

// Game functions
void playGame(vector<Card>& deck);

int main() {
    vector<Card> deck = createDeck();
    shuffleDeck(deck);

    playGame(deck);

    return 0;
}

vector<Card> createDeck() {
    vector<Card> deck;
    string suits[] = {"Hearts", "Diamonds", "Clubs", "Spades"};
    string ranks[] = {"2", "3", "4", "5", "6", "7", "8", "9", "10", "Jack", "Queen", "King", "Ace"};
    int values[] = {2, 3, 4, 5, 6, 7, 8, 9, 10, 10, 10, 10, 11};

    for (const auto& suit : suits) {
        for (size_t i = 0; i < 13; ++i) {
            deck.push_back({suit, ranks[i], values[i]});
        }
    }
    return deck;
}

void shuffleDeck(vector<Card>& deck) {
    shuffle(deck.begin(), deck.end(), default_random_engine(static_cast<unsigned>(time(0))));
}

Card drawCard(vector<Card>& deck) {
    if (deck.empty()) {
        cerr << "Error: Deck is empty, cannot draw a card!" << endl;
        exit(1);
    }
    Card card = deck.back();
    deck.pop_back();
    return card;
}

int calculateHandValue(const vector<Card>& hand, bool& isSoft) {
    int total = 0;
    int aceCount = 0;

    for (const auto& card : hand) {
        total += card.value;
        if (card.rank == "Ace") aceCount++;
    }

    isSoft = false;
    while (total > 21 && aceCount > 0) {
        total -= 10;
        aceCount--;
    }
    if (aceCount > 0) isSoft = true;

    return total;
}

void displayHand(const vector<Card>& hand, const string& name) {
    cout << name << " hand: ";
    for (const auto& card : hand) {
        cout << card.rank << " of " << card.suit << ", ";
    }
    cout << endl;
}

double calculateBustProbability(const vector<Card>& deck, int currentTotal) {
    if (deck.empty()) return 0.0;
    int bustCards = 0;
    for (const auto& card : deck) {
        if (currentTotal + card.value > 21) bustCards++;
    }
    return static_cast<double>(bustCards) / deck.size();
}

double calculateWinningProbability(const vector<Card>& deck, int playerTotal, int dealerVisible) {
    if (deck.empty()) return 0.0;

    int winCount = 0;
    int totalSimulations = 0;

    for (const auto& card : deck) {
        int dealerTotal = dealerVisible + card.value;

        // Adjust for Ace in dealer's initial hand
        if (dealerTotal > 21 && card.rank == "Ace") {
            dealerTotal -= 10;
        }

        // Dealer's fixed strategy: hit until reaching 17 or higher
        while (dealerTotal < 17) {
            for (const auto& nextCard : deck) {
                dealerTotal += nextCard.value;
                if (dealerTotal > 21 && nextCard.rank == "Ace") {
                    dealerTotal -= 10;
                }
                if (dealerTotal >= 17) break;
            }
        }

        // Compare dealer and player totals
        if (dealerTotal > 21 || playerTotal > dealerTotal) {
            winCount++;
        }

        totalSimulations++;
    }

    return static_cast<double>(winCount) / totalSimulations;
}

void playGame(vector<Card>& deck) {
    vector<Card> playerHand, dealerHand;
    playerHand.push_back(drawCard(deck));
    playerHand.push_back(drawCard(deck));
    dealerHand.push_back(drawCard(deck));
    dealerHand.push_back(drawCard(deck));

    displayHand(playerHand, "Player");
    displayHand({dealerHand[0]}, "Dealer");

    bool isSoft;
    int playerTotal = calculateHandValue(playerHand, isSoft);
    double bustProb = calculateBustProbability(deck, playerTotal);
    double winningProb = calculateWinningProbability(deck, playerTotal, dealerHand[0].value);

    cout << "Player total: " << playerTotal << " | Bust probability: " << bustProb * 100 << "%" << endl;
    cout << "Winning probability before hit: " << winningProb * 100 << "%" << endl;

    // Example: Player hits
    cout << "Player hits!" << endl;
    playerHand.push_back(drawCard(deck));
    playerTotal = calculateHandValue(playerHand, isSoft);
    bustProb = calculateBustProbability(deck, playerTotal);
    winningProb = calculateWinningProbability(deck, playerTotal, dealerHand[0].value);

    displayHand(playerHand, "Player");
    cout << "Player total: " << playerTotal << " | Bust probability: " << bustProb * 100 << "%" << endl;
    cout << "Winning probability after hit: " << winningProb * 100 << "%" << endl;
}

