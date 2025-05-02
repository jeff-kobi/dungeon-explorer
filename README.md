using System;
using System.Collections.Generic;
using System.Linq;
using DungeonExplorer.Tests;

namespace DungeonExplorer
{
    public class Room
    {
        public string Description { get; set; }
        public List<Creature> Monsters { get; set; } = new List<Creature>();
        public Dictionary<string, Room> Exits { get; set; } = new Dictionary<string, Room>();
        public List<Item> Items { get; set; } = new List<Item>();

        public Room(string description)
        {
            Description = description;
        }

        public void AddExit(string direction, Room destination)
        {
            Exits[direction.ToLower()] = destination;
        }

        public void AddItem(Item item)
        {
            Items.Add(item);
            Console.WriteLine($"Added {item.Name} to the room.");
        }

        



        public Item TakeItem(string itemName)
        {
            var item = Items.FirstOrDefault(i => i.Name.Equals(itemName, StringComparison.OrdinalIgnoreCase));
            if (item != null)
            {
                Items.Remove(item);
                return item;
            }
            return null;
        }
    }
}
 
