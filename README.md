# Chess-system
A Chess system mostly in Java, The system exposes a REST API in which two neural networks created in Python play against each other, learning from each other's mistakes.

The AIs are initially trained to learn the basics of the board game and as soon as they acquire game fluency, they are allowed to compete with each other.


                Java Core Engine                
                                              
  model/          Pure domain objects           
  ├── Board       8×8 board state                
  ├── Move        Move representation            
  ├── Position    Algebraic ↔ row/col            
  └── pieces/     King, Queen, Rook, ...         
                                                 
  engine/         Game logic                     
  ├── GameEngine  Move history, status, FEN      
  ├── MoveValidator  Legal move generation       
  └── FenSerializer  FEN export/import           
                                                 
  server/                                        
  └── ChessApiServer  REST API (port 8080)       

                HTTP / JSON
white_ai.py: RandomAgent 
black_ai.py: GreedyAgent 



              THE REST API 

Method           EndPoint         Description

GET              /state         Full game state; FEN, status, legal moves
POST             /move          Apply a move {"move": "e2e4"}
GET              /moves         Legal moves for the current turn
POST             /reset         Reset to starting moves


An example response for /state
{
  "fen": "rnbqkbnr/pppppppp/8/8/4P3/8/PPPP1PPP/RNBQKBNR b KQkq e3 0 1",
  "status": "IN_PROGRESS",
  "turn": "BLACK",
  "fullMoveNumber": 1,
  "halfMoveClock": 0,
  "legalMoves": ["e7e5", "e7e6", "d7d5", "..."]
}

*QUICK START*

*1. Build & run the Java Engine:*

mvn package -q
java -jar target/chess-ai-system-1.0.0.jar
# Server starts on http://localhost:8080

*2. Install Python Dependencies*

   cd python-ai
pip install -r requirements.txt

*3a. Run both AI's together*

cd python-ai/agents
python orchestrator.py

*3b. Run AI's in separate terminals*
# Terminal 2
python python-ai/agents/white_ai.py
# Terminal 3 
python python-ai/agents/black_ai.py

*Implementing your own AI:*
Subclass BaseChessAgent and implement choose_move:

from base_agent import BaseChessAgent
import random

class MyCustomAI(BaseChessAgent):
    def choose_move(self, state: dict, legal_moves: list[str]) -> str:
        # state["fen"]         — current FEN string
        # state["legalMoves"]  — list of legal moves in algebraic notation
        # Your logic here:
        return random.choice(legal_moves)

if __name__ == "__main__":
    agent = MyCustomAI(color="WHITE")
    agent.play()

Running tests
mvn test in cmd/bash/powershell


chess-ai-system/
├── pom.xml
├── src/
│   ├── main/java/com/chess/
│   │   ├── Main.java
│   │   ├── model/
│   │   │   ├── Board.java
│   │   │   ├── Color.java
│   │   │   ├── GameStatus.java
│   │   │   ├── Move.java
│   │   │   ├── MoveType.java
│   │   │   ├── PieceType.java
│   │   │   ├── Position.java
│   │   │   └── pieces/
│   │   │       ├── Piece.java       ← abstract base
│   │   │       ├── King.java
│   │   │       ├── Pawn.java
│   │   │       └── SlidingPieces.java  (Queen, Rook, Bishop, Knight)
│   │   ├── engine/
│   │   │   ├── GameEngine.java
│   │   │   ├── MoveValidator.java
│   │   │   ├── FenSerializer.java
│   │   │   ├── PieceFactory.java
│   │   │   └── IllegalMoveException.java
│   │   └── server/
│   │       └── ChessApiServer.java
│   └── test/java/com/chess/
│       └── GameEngineTest.java
└── python-ai/
    ├── requirements.txt
    └── agents/
        ├── base_agent.py     ← abstract base, all HTTP logic here
        ├── white_ai.py       ← Random agent
        ├── black_ai.py       ← Greedy capture agent
        └── orchestrator.py   ← runs both AIs in one process

        

*Extending the Java Engine* 
FEN import (for loading custom positions)
Implement FenSerializer.fromFen() — the stub is already in place.
Castling rights tracking
Currently simplified to KQkq. Add a CastlingRights object to Board and update it in MoveValidator.applyMoveToBoard().
Move history / PGN export
GameEngine.getMoveHistory() already exposes the move list — add a PgnExporter class.



*Tech Stack*

Java 17 — engine + REST server
Jackson — JSON serialization
JUnit 5 — unit tests
Python 3.10+ — AI agents
requests — HTTP client for Python agents
