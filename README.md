# snakes1

using System;
using System.Collections.Generic;
using System.Drawing;
using System.Linq;
using System.Windows.Forms;

namespace Snakes
{
    public partial class GameForm : Form
    {
        private const int BlockSize = 20;

        private readonly List<Point> _snake = new List<Point>();
        private Point _food;

        private enum Direction
        {
            Up,
            Down,
            Left,
            Right
        }

        private Direction _currentDirection = Direction.Right;

        public GameForm()
        {
            InitializeComponent();

            // Set the form properties
            Width = 600;
            Height = 400;
            Text = "Snakes";

            // Set up the game
            ResetGame();

            // Start the game loop
            var timer = new Timer { Interval = 100 };
            timer.Tick += (sender, args) => GameLoop();
            timer.Start();
        }

        private void ResetGame()
        {
            // Clear the snake
            _snake.Clear();

            // Set the starting position of the snake
            var head = new Point(10, 10);
            _snake.Add(head);

            // Set the position of the food
            _food = GetRandomFoodPosition();
        }

        private Point GetRandomFoodPosition()
        {
            var rand = new Random();
            var x = rand.Next(0, Width / BlockSize) * BlockSize;
            var y = rand.Next(0, Height / BlockSize) * BlockSize;
            return new Point(x, y);
        }

        private void GameLoop()
        {
            // Move the snake
            var head = _snake.Last();
            switch (_currentDirection)
            {
                case Direction.Up:
                    head = new Point(head.X, head.Y - BlockSize);
                    break;
                case Direction.Down:
                    head = new Point(head.X, head.Y + BlockSize);
                    break;
                case Direction.Left:
                    head = new Point(head.X - BlockSize, head.Y);
                    break;
                case Direction.Right:
                    head = new Point(head.X + BlockSize, head.Y);
                    break;
            }

            // Check if the snake has hit a wall
            if (head.X < 0 || head.Y < 0 || head.X >= Width || head.Y >= Height)
            {
                ResetGame();
                return;
            }

            // Check if the snake has hit itself
            if (_snake.Any(block => block == head))
            {
                ResetGame();
                return;
            }

            // Check if the snake has eaten the food
            if (head == _food)
            {
                // Add a new block to the snake
                _snake.Add(head);

                // Set the position of the food
                _food = GetRandomFoodPosition();
            }
            else
            {
                // Move the snake
                _snake.RemoveAt(0);
                _snake.Add(head);
            }

            // Redraw the game
            Invalidate();
        }

        private void GameForm_Paint(object sender, PaintEventArgs e)
        {
            // Draw the snake
            foreach (var block in _snake)
            {
                var rect = new Rectangle(block.X, block.Y, BlockSize, BlockSize);
                e.Graphics.FillRectangle(Brushes.Green, rect);
            }

            // Draw the food
            var foodRect = new Rectangle(_food.X, _food.Y, BlockSize, BlockSize);
            e.Graphics.FillEllipse(Brushes.Red, foodRect);
        }

        private void GameForm_KeyDown(object sender, KeyEventArgs e)
        {
            switch (e.KeyCode)
            {
                case Keys.Up:
