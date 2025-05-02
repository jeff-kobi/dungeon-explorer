using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Runtime.InteropServices;
using DungeonExplorer;
using DungeonExplorer.Tests;

namespace DungeonExplorer
{

    public class Potion : Item
    {
        public int HealAmount { get; set; }

        public Potion(string name, int healAmount, string desc, bool useable) : base(name, desc, useable)
        {
            HealAmount = healAmount;
        }

        public override void Use(Player player)
        {
            player.Heal(HealAmount);
            Console.WriteLine($"{player.Name} drinks {Name} and heals {HealAmount} HP.");
        }
    }

    // Monster classes
    public class Dragon : Creature
    {
        public Dragon() : base("Dragon", 150, 30) { }

        public override void Attack(Player target)
        {
            Console.WriteLine("Dragon breathes fire!");
            target.TakeDamage(AttackPower);
        }
    }

    public class Witch : Creature
    {
        public Witch() : base("Witch", 90, 15) { }

        public override void Attack(Player target)
        {
            Console.WriteLine("Witch casts a dark spell!");
            target.TakeDamage(AttackPower);
        }
    }

    public class Goblin : Creature
    {
        public Goblin() : base("Goblin", 70, 10) { }

        public override void Attack(Player target)
        {
            Console.WriteLine("Goblin slashes wildly!");
            target.TakeDamage(AttackPower);
        }
    }

    public class Gremlin : Creature
    {
        public Gremlin() : base("Gremlin", 60, 12) { }

        public override void Attack(Player target)
        {
            Console.WriteLine("Gremlin bites sneakily!");
            target.TakeDamage(AttackPower);
        }
    }

    public class Ghost : Creature
    {
        public Ghost() : base("Ghost", 80, 13) { }

        public override void Attack(Player target)
        {
            Console.WriteLine("Ghost haunts and chills!");
            target.TakeDamage(AttackPower);
        }
    }

    public class Vampire : Creature
    {
        public Vampire() : base("Vampire", 100, 18) { }

        public override void Attack(Player target)
        {
            Console.WriteLine("Vampire drains life!");
            target.TakeDamage(AttackPower);
            // Vampire heals himself a bit
            Health = Math.Min(Health + 5, 100);  // Cap at original health
            Console.WriteLine($"Vampire drains 5 life! Current HP: {Health}");
        }
    }

    public class Werewolf : Creature
    {
        public Werewolf() : base("Werewolf", 120, 20) { }

        public override void Attack(Player target)
        {
            Console.WriteLine("Werewolf claws viciously!");
            target.TakeDamage(AttackPower);
        }
    }

    public class Game
    {
        private Player _player;
        private bool _isRunning;
        private List<Room> _rooms;

        public Game()
        {
            _player = new Player("Hero", 100, 15);
            _isRunning = true;
            _rooms = new List<Room>();
        }

        public void InitializeGame()
        {
            // Create the 7 rooms
            Room forest = new Room("A haunted forest echoing with howls");
            Room graveyard = new Room("A foggy graveyard with eerie silence");
            Room castle = new Room("An abandoned castle with traps");
            Room swamp = new Room("A misty swamp full of croaks");
            Room library = new Room("A cursed library with floating books");
            Room dungeon = new Room("A blood-soaked dungeon of legends");

            _rooms.Add(forest);
            _rooms.Add(graveyard);
            _rooms.Add(castle);
            _rooms.Add(swamp);
            _rooms.Add(library);
            _rooms.Add(dungeon);

            // Connect rooms (bidirectional)
            CreateConnection(forest, "south", graveyard);
            CreateConnection(graveyard, "west", castle);
            CreateConnection(castle, "north", swamp);
            CreateConnection(swamp, "east", library);
            CreateConnection(library, "south", dungeon);
            // Add more connections for better exploration
            CreateConnection(dungeon, "west", castle);
            CreateConnection(swamp, "south", graveyard);

            // Add monsters to rooms
            forest.Monsters.Add(new Werewolf());
            graveyard.Monsters.Add(new Ghost());
            castle.Monsters.Add(new Vampire());
            swamp.Monsters.Add(new Gremlin());
            library.Monsters.Add(new Goblin());
            dungeon.Monsters.Add(new Witch());

            // Add items to player's inventory
            _player.addItem(new Weapon("Hammer of Thor", 25, "The hammer", true));
            _player.addItem(new Weapon("Arthur's Sword", 20, "Common", true));

            // Add items to rooms
            library.Items.Add(new Potion("Super Strength Potion", 0, "Increase health", true));
            graveyard.Items.Add(new Potion("Health Potion", 25, "Regen", true));

            // Set starting room
            _player.CurrentRoom = forest;
        }

        private void CreateConnection(Room from, string direction, Room to)
        {
            // Determine opposite direction
            string opposite = GetOppositeDirection(direction);

            // Create bidirectional connection
            from.AddExit(direction, to);
            to.AddExit(opposite, from);
        }

        private string GetOppositeDirection(string direction)
        {

            if (direction == "north")
            {
                return "south";
            }
            else if (direction == "south")
            {
                return "north";
            }
            else if (direction == "east")
            {
                return "west";
            }
            else if (direction == "west")
            {
                return "east";
            }
            else if (direction == "up")
            {
                return "down";
            }
            else if (direction == "dowm")
            {
                return "up";
            }
            return "";
        }
        public void Move(string direction)
        {
            if (_rooms == null)
            {
                Console.WriteLine("Error: Player is not in any room!");
                return;
            }

            if (string.IsNullOrWhiteSpace(direction))
            {
                Console.WriteLine("Please specify a direction to move.");
                _player.ShowAvailableDirections();
                return;
            }

            if (_player.CurrentRoom.Exits.TryGetValue(direction.ToLower(), out var nextRoom))
            {
                _player.CurrentRoom = nextRoom;
                Console.WriteLine($"You move to: {_player.CurrentRoom.Description}");

                if (_player.CurrentRoom.Monsters.Any(m => m.isAlive()))
                {
                    Console.WriteLine("You encounter:");
                    foreach (var monster in _player.CurrentRoom.Monsters.Where(m => m.isAlive()))
                    {
                        Console.WriteLine($"- A {monster.Name} (HP: {monster.Health}, Attack: {monster.AttackPower})");
                    }
                }
                else
                {
                    Console.WriteLine("This area appears to be safe... for now.");
                }

                _player.ShowAvailableDirections();
            }
            else
            {
                Console.WriteLine("You can't go that way!");
                _player.ShowAvailableDirections();
            }
        }


        public void Run()
        {
            InitializeGame();

            Console.WriteLine("==============================");
            Console.WriteLine("Welcome to Dungeon Explorer!");
            Console.WriteLine("==============================");
            Console.WriteLine("You awaken in a mysterious realm...");
            Console.WriteLine(_player.CurrentRoom?.Description);
            _player.ShowAvailableDirections();
            Console.WriteLine("\nCommands:");
            Console.WriteLine("- move [direction]: Move in a direction (north, south, east, west)");
            Console.WriteLine("- look: Examine your surroundings");
            Console.WriteLine("- take [item]: Pick up an item");
            Console.WriteLine("- use [item]: Use an item from your inventory");
            Console.WriteLine("- attack: Fight a monster in the room");
            Console.WriteLine("- stats: Show your statistics and inventory");
            Console.WriteLine("- map: Display explored areas");
            Console.WriteLine("- help: Show this help message");
            Console.WriteLine("- quit: Exit the game");

            while (_isRunning)
            {
                Console.Write("\n> ");
                string input = Console.ReadLine()?.Trim().ToLower();

                if (string.IsNullOrWhiteSpace(input))
                {
                    continue;
                }

                string[] parts = input.Split(' ', ' ');
                string command = parts[0];
                string argument = parts.Length > 1 ? parts[1] : string.Empty;

                ProcessCommand(command, argument);

                // Check game over condition
                if (!(_player.isAlive()))
                {
                    Console.WriteLine("\n╔════════════════════════════════════╗");
                    Console.WriteLine("║          G A M E   O V E R          ║");
                    Console.WriteLine("╚════════════════════════════════════╝");
                    _isRunning = false;
                }
            }

            Console.WriteLine("\nThanks for playing Dungeon Explorer!");
        }

        private void ProcessCommand(string command, string argument)
        {
            switch (command)
            {
                case "move":
                case "go":
                    Move(argument);
                    break;

                case "look":
                    if (_player.CurrentRoom != null)
                    {
                        Console.WriteLine($"\n{_player.CurrentRoom.Description}");

                        // List monsters
                        var liveMonsters = _player.CurrentRoom.Monsters.Where(m => m.isAlive());
                        if (liveMonsters.Any())
                        {
                            Console.WriteLine("\nMonsters present:");

                            foreach (var _monster in liveMonsters)
                            {
                                Console.WriteLine($"- {_monster.Name} (HP: {_monster.Health}, Attack: {_monster.AttackPower})");
                            }
                        }

                        // List items
                        if (_player.CurrentRoom.Items.Any())
                        {
                            Console.WriteLine("\nItems in the area:");
                            foreach (var item in _player.CurrentRoom.Items)
                            {
                                if (item is Weapon weapon)
                                    Console.WriteLine($"- {weapon.Name} (Weapon, +{weapon.BaseDamage} damage)");
                                else if (item is Potion potion)
                                    Console.WriteLine($"- {potion.Name} (Potion, +{potion.HealAmount} health)");
                                else
                                    Console.WriteLine($"- {item.Name}");
                            }
                        }

                        _player.ShowAvailableDirections();
                    }
                    break;

                case "take":
                    if (string.IsNullOrWhiteSpace(argument))
                    {
                        Console.WriteLine("What do you want to take?");
                        break;
                    }

                    if (_player.CurrentRoom != null)
                    {
                        Item item = _player.CurrentRoom.TakeItem(argument);
                        if (item != null)
                        {
                            _player.addItem(item);
                        }
                        else
                        {
                            Console.WriteLine($"There is no '{argument}' to take.");
                        }
                    }
                    break;

                case "use":
                    _player.UseItem(argument);
                    break;

                case "attack":
                    if (_player.CurrentRoom == null)
                    {
                        Console.WriteLine("Error: Player is not in any room!");
                        break;
                    }

                    var monster = _player.CurrentRoom.Monsters.FirstOrDefault(m => m.isAlive());
                    if (monster != null)
                    {
                        // Combat system
                        _player.Attack(monster);

                        if (monster.isAlive())
                        {
                            // Monster counterattacks
                            monster.Attack(_player);
                        }
                        else
                        {
                            Console.WriteLine($"You defeated the {monster.Name}!");
                        }
                    }
                    else
                    {
                        Console.WriteLine("There are no monsters to fight here.");
                    }
                    break;

                case "stats":
                    _player.ShowStats();
                    break;

                case "map":
                    _player.ShowMap();
                    break;

                case "help":
                    Console.WriteLine("\nCommands:");
                    Console.WriteLine("- move [direction]: Move in a direction (north, south, east, west)");
                    Console.WriteLine("- look: Examine your surroundings");
                    Console.WriteLine("- take [item]: Pick up an item");
                    Console.WriteLine("- use [item]: Use an item from your inventory");
                    Console.WriteLine("- attack: Fight a monster in the room");
                    Console.WriteLine("- stats: Show your statistics and inventory");
                    Console.WriteLine("- map: Display explored areas");
                    Console.WriteLine("- help: Show this help message");
                    Console.WriteLine("- quit: Exit the game");
                    break;

                case "quit":
                case "exit":
                    Console.Write("Are you sure you want to quit? (y/n): ");
                    string response = Console.ReadLine()?.Trim().ToLower();
                    if (response == "y" || response == "yes")
                    {
                        _isRunning = false;
                    }
                    break;

                default:
                    Console.WriteLine("Unknown command. Type 'help' for a list of commands.");
                    break;
            }
        }
    }


        



       
