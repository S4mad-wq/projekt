using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        BlackjackGame game = new BlackjackGame();
        game.Start();
    }
}

class BlackjackGame
{
    private Deck deck;
    private Player player;
    private Player dealer;

    public BlackjackGame()
    {
        deck = new Deck();
        player = new Player("Player");
        dealer = new Player("Dealer");
    }

    public void Start()
    {
        Console.WriteLine("Welcome to Blackjack!");
        deck.Shuffle();

        // Раздача карт
        player.AddCard(deck.DealCard());
        player.AddCard(deck.DealCard());
        dealer.AddCard(deck.DealCard());
        dealer.AddCard(deck.DealCard());

        // Игровой процесс
        while (true)
        {
            Console.WriteLine($"\n{player.Name}'s hand: {player.ShowHand()} (Total: {player.GetTotalValue()})");
            Console.WriteLine($"{dealer.Name}'s visible card: {dealer.ShowHand()[0]}");

            if (player.GetTotalValue() == 21)
            {
                Console.WriteLine("Blackjack! You win!");
                return;
            }

            Console.Write("Do you want to hit (h) or stand (s)? ");
            string choice = Console.ReadLine();

            if (choice.ToLower() == "h")
            {
                player.AddCard(deck.DealCard());
                if (player.GetTotalValue() > 21)
                {
                    Console.WriteLine($"Busted! Your hand: {player.ShowHand()} (Total: {player.GetTotalValue()})");
                    return;
                }
            }
            else if (choice.ToLower() == "s")
            {
                break;
            }
        }

        // Дилерская игра
        while (dealer.GetTotalValue() < 17)
        {
            dealer.AddCard(deck.DealCard());
        }

        // Результаты
        Console.WriteLine($"\n{dealer.Name}'s hand: {dealer.ShowHand()} (Total: {dealer.GetTotalValue()})");
        DetermineWinner();
    }

    private void DetermineWinner()
    {
        int playerTotal = player.GetTotalValue();
        int dealerTotal = dealer.GetTotalValue();

        if (dealerTotal > 21 || playerTotal > dealerTotal)
        {
            Console.WriteLine("You win!");
        }
        else if (playerTotal < dealerTotal)
        {
            Console.WriteLine("Dealer wins!");
        }
        else
        {
            Console.WriteLine("It's a tie!");
        }
    }
}

class Player
{
    public string Name { get; }
    private List<Card> hand;

    public Player(string name)
    {
        Name = name;
        hand = new List<Card>();
    }

    public void AddCard(Card card)
    {
        hand.Add(card);
    }

    public List<Card> ShowHand()
    {
        return hand;
    }

    public int GetTotalValue()
    {
        int total = 0;
        int aces = 0;

        foreach (var card in hand)
        {
            total += card.Value;
            if (card.Rank == Rank.Ace)
                aces++;
        }

        while (total > 21 && aces > 0)
        {
            total -= 10; // Превращаем туз из 11 в 1
            aces--;
        }

        return total;
    }
}

class Deck
{
    private List<Card> cards;
    private Random random;

    public Deck()
    {
        cards = new List<Card>();
        foreach (Rank rank in Enum.GetValues(typeof(Rank)))
        {
            foreach (Suit suit in Enum.GetValues(typeof(Suit)))
            {
                cards.Add(new Card(rank, suit));
            }
        }
        random = new Random();
    }

    public void Shuffle()
    {
        for (int i = 0; i < cards.Count; i++)
        {
            int r = random.Next(i, cards.Count);
            Card temp = cards[i];
            cards[i] = cards[r];
            cards[r] = temp;
        }
        public Card DealCard()
    {
        if (cards.Count == 0)
            throw new InvalidOperationException("No cards left in the deck.");

        Card card = cards[cards.Count - 1];
        cards.RemoveAt(cards.Count - 1);
        return card;
    }
}

class Card
{
    public Rank Rank { get; }
    public Suit Suit { get; }
    
    public int Value => Rank == Rank.Ace ? 11 : (int)Rank > 10 ? 10 : (int)Rank;

    public Card(Rank rank, Suit suit)
    {
        Rank = rank;
        Suit = suit;
    }

    public override string ToString()
    {
        return $"{Rank} of {Suit}";
    }
}

enum Rank
{
    Two = 2,
    Three,
    Four,
    Five,
    Six,
    Seven,
    Eight,
    Nine,
    Ten,
    Jack = 10,
    Queen = 10,
    King = 10,
    Ace = 11
}

enum Suit
{
    Hearts,
    Diamonds,
    Clubs,
    Spades
}
