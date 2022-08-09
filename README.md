# Snake-Plus-Plus
Snake game created in C++; Only works on Windows because of the conio.h header file;
When compiling, use C++14 as a minimum and C++20 as recommended;

Reason for this being made{ Experience, wanting to create this game without looking at internet ones and succeded, and after finishing it and looking at the ones already made I see a lot of magic numbers and overall the code isnt readable, this code should be easily readable by an experienced C++ developer };



I tried to embed as many features of C++ into this project and succeded with the following:

  - Lambda Functions

  - Memory-constants integers on all devices

  - Preprocesor definitions

  - constexpr

  - Structs

  - Enums

  - Threads

  - Pointers

  - Rand (used it because there is no need for security in a local snake game)



Overall im really proud of how it turned out :)

```Preview```
>![game](prewiew.png)


>![C++](https://img.shields.io/badge/c++-%2300599C.svg?style=for-the-badge&logo=c%2B%2B&logoColor=white)

```
#include <iostream>
#include <thread>
#include <conio.h>
#include <cstdint>
using std::cin;
using std::cout;
using std::rand;
using std::srand;
using std::string;
using std::thread;
using std::chrono::milliseconds;
using std::chrono::seconds;
constexpr uint8_t MAP_ROWS{ 7 }; 
constexpr uint8_t MAP_LIMITS{ 2 };
constexpr uint8_t MAP_COLUMNS{ 20 };
constexpr uint8_t MAP_START{ 1 };
constexpr uint8_t EMPTY_SPACE{ 0 };
constexpr uint8_t SNAKE_BODY{ 1 };
constexpr uint8_t SNAKE_HEAD{ 2 };
constexpr uint8_t MAP_EMPTY_SPACE{ (MAP_COLUMNS - MAP_LIMITS) * (MAP_ROWS - MAP_LIMITS) };
constexpr uint8_t SCORE_OFFSET{ 4 }; // The value of this must be the same as the value of the snake's length

#define Y_RAND rand()%(MAP_ROWS-MAP_LIMITS)+MAP_START
#define X_RAND rand()%(MAP_COLUMNS-MAP_LIMITS)+MAP_START

// Game map
string map[MAP_ROWS] = {
    "--------------------",
    "|                  |",
    "|                  |",
    "|                  |",
    "|                  |",
    "|                  |",
    "--------------------" };

// Direction enum
enum Direction
{
    LEFT,
    RIGHT,
    UP,
    DOWN
};


// Snake/Fruit structs
struct Snake
{
    uint8_t snakePos[MAP_ROWS][MAP_COLUMNS];
    size_t length;
    uint8_t headPos_X;
    uint8_t headPos_Y;
    uint8_t tailPos_X[MAP_COLUMNS * MAP_ROWS];
    uint8_t tailPos_Y[MAP_ROWS * MAP_COLUMNS];
    unsigned char bodyChar;
    unsigned char headChar;
    bool gameLost;
    bool gameWon;
};
struct Fruit
{
    size_t X_POS;
    size_t Y_POS;
    unsigned char fChar;
    bool wasEaten;
};

// display
void display(Snake snake, Fruit fruit)
{
    system("cls");
    cout << "--------------------\n";
    for (size_t Y_AXIS{ 1 }; Y_AXIS < MAP_ROWS - 1; Y_AXIS++) // -1 because at last we can print the entire line without wasting performance
    {
        for (size_t X_AXIS{ 0 }; X_AXIS < MAP_COLUMNS; X_AXIS++)
        {
            // Snake display
            if (snake.snakePos[Y_AXIS][X_AXIS] == SNAKE_BODY)
            {
                cout << snake.bodyChar;
                continue;
            }
            else if (snake.snakePos[Y_AXIS][X_AXIS] == SNAKE_HEAD)
            {
                cout << snake.headChar;
                continue;
            }

            // Fruit display
            if (fruit.X_POS == X_AXIS && fruit.Y_POS == Y_AXIS)
            {
                cout << fruit.fChar;
                continue;
            }

            cout << map[Y_AXIS][X_AXIS];
        }
        cout << '\n';
    }
    cout << "--------------------\n";
    cout << "\t    ";

    if (snake.gameWon)
    {
        cout << "GAME WON: " << snake.length - SCORE_OFFSET << '\n';
        std::this_thread::sleep_for(seconds(600000000));
    }

    if (snake.gameLost)
    {
        cout << "GAME OVER: " << snake.length - SCORE_OFFSET << '\n';
        std::this_thread::sleep_for(seconds(600000000));
    }
    cout << "Score: " << snake.length - SCORE_OFFSET << '\n';
}

// inputHandler
void inputHandler(Direction* direction, Snake* snake)
{
    unsigned char inp = _getch();
    switch (inp)
    {
    case 'a':
        *direction = Direction::LEFT;
        snake->headChar = '<';
        break;
    case 'd':
        *direction = Direction::RIGHT;
        snake->headChar = '>';
        break;
    case 'w':
        *direction = Direction::UP;
        snake->headChar = '^';
        break;
    case 's':
        *direction = Direction::DOWN;
        snake->headChar = 'v';
        break;
    default:
        break;
    }
}

// movementHandler
void movementHandler(Snake* snake, Direction direction)
{
    size_t nextPosition_X{ snake->headPos_X };
    size_t nextPosition_Y{ snake->headPos_Y };
    size_t lastTailIndex{ static_cast<size_t>(snake->length - 2) }; // 2 accounts for: 1) array index; 2) the fact that the head doesnt count
    switch (direction)
    {
    case Direction::LEFT:
        snake->headPos_X--;
        break;
    case Direction::RIGHT:
        snake->headPos_X++;
        break;
    case Direction::UP:
        snake->headPos_Y--;
        break;
    case Direction::DOWN:
        snake->headPos_Y++;
        break;
    default:
        break;
    }

    for (size_t ROWS{ 0 }; ROWS < MAP_ROWS; ROWS++)
    {
        for (size_t COLUMNS{ 0 }; COLUMNS < MAP_COLUMNS; COLUMNS++)
        {
            snake->snakePos[ROWS][COLUMNS] = EMPTY_SPACE;
        }
    }

    for (size_t index{ 0 }; index < lastTailIndex; index++)
    {
        snake->tailPos_X[index] = snake->tailPos_X[index + 1];
        snake->tailPos_Y[index] = snake->tailPos_Y[index + 1];
        snake->snakePos[snake->tailPos_Y[index]][snake->tailPos_X[index]] = SNAKE_BODY;
    }
    snake->tailPos_X[lastTailIndex] = nextPosition_X;
    snake->tailPos_Y[lastTailIndex] = nextPosition_Y;
    snake->snakePos[nextPosition_Y][nextPosition_X] = SNAKE_BODY;
    snake->snakePos[snake->headPos_Y][snake->headPos_X] = SNAKE_HEAD;
}

// fruitHandler
void fruitHandler(Snake* snake, Fruit* fruit)
{
    if (!(snake->headPos_Y == fruit->Y_POS && snake->headPos_X == fruit->X_POS)) { return; }

    auto checkFruit = [=]() -> bool
    {
        if (snake->headPos_Y == fruit->Y_POS && snake->headPos_X == fruit->X_POS)
        {
            return true;
        }
        for (size_t index{ 0 }; index < snake->length; index++)
        {
            if (snake->tailPos_X[index] == fruit->X_POS && snake->tailPos_Y[index] == fruit->Y_POS)
            {
                return true;
            }
        }
        snake->length++;
        return false;
    };

    do
    {
        fruit->Y_POS = Y_RAND;
        fruit->X_POS = X_RAND;
    } while (checkFruit());
}

// gameChecker
void gameChecker(Snake* snake)
{
    auto snakeCollision = [=]() -> bool
    {
        for (size_t index{ 0 }; index < snake->length; index++)
        {
            if (snake->headPos_X == snake->tailPos_X[index] && snake->headPos_Y == snake->tailPos_Y[index])
            {
                return true;
            }
        }
        return false;
    };

    if (snake->length == MAP_EMPTY_SPACE)
    {
        snake->gameWon = true;
        return;
    }

    if ((snake->headPos_X < MAP_START || snake->headPos_X > MAP_COLUMNS - MAP_LIMITS || snake->headPos_Y < MAP_START || snake->headPos_Y > MAP_ROWS - MAP_LIMITS) || (snakeCollision()))
    {
        snake->gameLost = true;
    }
}

int main()
{
    // Initialization
    srand(time(NULL));
    Fruit fruit{};
    fruit.X_POS = 15;
    fruit.Y_POS = 1;
    fruit.fChar = '#';

    Snake snake{};
    snake.gameLost = false;
    snake.gameWon = false;

    snake.bodyChar = static_cast<char>(248);
    snake.headChar = '>';

    // Snake starting point
    snake.tailPos_X[0] = 1;
    snake.tailPos_Y[0] = 1;
    snake.tailPos_X[1] = 2;
    snake.tailPos_Y[1] = 1;
    snake.tailPos_X[2] = 3;
    snake.tailPos_Y[2] = 1;

    snake.snakePos[1][1] = SNAKE_BODY;
    snake.snakePos[1][2] = SNAKE_BODY;
    snake.snakePos[1][3] = SNAKE_BODY;
    snake.snakePos[1][4] = SNAKE_HEAD;

    snake.length = 4;
    snake.headPos_X = 4;
    snake.headPos_Y = 1;

    // Direction
    Direction direction{ Direction::RIGHT };

    // Main loop
    while (true)
    {
        display(snake, fruit);
        std::this_thread::sleep_for(milliseconds(100));
        if (_kbhit())
        {
            inputHandler(&direction, &snake);
        }
        movementHandler(&snake, direction);
        fruitHandler(&snake, &fruit);
        gameChecker(&snake);
    }

    return 0;
}
```

