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
          ┌──────────┴──────────┐
          │                     │
   ┌──────▼──────┐       ┌──────▼──────┐
     white_ai.py           black_ai.py 
     RandomAgent           GreedyAgent 
   └─────────────┘       └─────────────┘



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


