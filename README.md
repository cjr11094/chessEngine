chessEngine
===========

My name is Conner Reilly and I'm a student at Amherst College. I've created this chess engine using alpha-beta pruning and would like to continue making adjustments.
=========== AlphaBetaChess.java


public class AlphaBetaChess {

	
	String chessboard[][];
	
	public int kingRowC, kingColC, kingRowL, kingColL;
	
	public int depth = 4;
	
	public String moveToReturn;
	
	public int bishopCapValue;
	public int knightCapValue;
	public int rookCapValue;
	public int queenCapValue;
	public int kingCapValue;
	public int pawnCapValue;
	
	public AlphaBetaChess(String board, int bishopCapValue, int knightCapValue, int rookCapValue, int queenCapValue, int kingCapValue, int pawnCapValue){
		this.bishopCapValue = bishopCapValue;
		this.knightCapValue = knightCapValue;
		this.rookCapValue = rookCapValue;
		this.queenCapValue = queenCapValue;
		this.kingCapValue = kingCapValue;
		this.pawnCapValue = pawnCapValue;
		
		int counter = 0;
		String tempchessboard[][] = new String[8][8];
		for( int i = 0 ; i < 8 ; i++){
			for( int j = 0 ; j < 8 ; j++){
				tempchessboard[i][j]=board.substring(counter,counter+1);
				counter++;
			}
		}
		for(int i = 0; i < 8 ; i++){
			for(int j = 0; j < 8 ; j++){
				if(tempchessboard[i][j].equals("A")){
					kingRowC=i;
					kingColC=j;
				}
				if(tempchessboard[i][j].equals("a")){
					kingRowL=i;
					kingColL=j;
				}
			}
		}
		this.chessboard = tempchessboard;
		
		String move = alphabeta("",Integer.MIN_VALUE,Integer.MAX_VALUE,depth,0);

		moveToReturn = move;
	}
	
	public void printBoard(){
		for(int i = 0; i < 8 ; i++){
			for(int j = 0; j < 8 ; j++){
				System.out.print(chessboard[i][j]+" ");
			}
			System.out.println();
		}
	}
	
	// this is going to be my alpha-beta pruning method..."String move" will represent the move
	// being made, and max will represent which player we are evaluating the board for
	public String alphabeta (String move, int alpha, int beta, int depth, int max){
		// I'm going to encode the score as being move,value (so e.g. 3444e4000)
		// where 34 is original row and col, 44 is new row and col, and 4000 is the score
		String possMoves = possibleMoves();

		// base case is when we are at depth 0 or there are no moves remaining
		if(depth==0 || possMoves.length()==0){
			return move+( ((max*2)-1)*(heuristic(possMoves.length(), depth)) );
		}
		// flip max (when max==1 we are at a max node, when max==0 we are at a min node)
		max = 1-max;
		for(int i = 0; i < possMoves.length() ; i+=5){
			movePiece(possMoves.substring(i,i+5));
			rotateBoard();
			String moveandscore = alphabeta(possMoves.substring(i,i+5),alpha,beta,depth-1,max);
			int value = Integer.parseInt(moveandscore.substring(5));
			rotateBoard();
			unmovePiece(possMoves.substring(i,i+5));
			if(max==0){
				if(value<=beta){
					beta = value;
					if(depth==this.depth){
						move = moveandscore.substring(0,5);
					}
				}
			} else {
				if(value>alpha){
					alpha = value;
					if(depth==this.depth){
						move = moveandscore.substring(0,5);
					}
				}
			}
			if(alpha>=beta){
				if(max == 0){
					return move+beta;
				} else {
					return move+alpha;
				}
			}
		}
		if(max == 0){
			return move+beta;
		} else {
			return move+alpha;
		}
	}
	
	// if I get the time, I would like to implement a technique called "progressive deepening"
	// basically what happens with this technique is that we call alpha-beta at each depth
	// and calculate a move at each depth...I found out about this here
	// http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-034-artificial-intelligence-fall-2010/lecture-videos/lecture-6-search-games-minimax-and-alpha-beta/
	// The mathematical explanation is ~35 minutes in
	
	// as far as actual implementation goes, I got to reading about transposition tables
	// I would like to implement the iterative deepening at some point
	public void searchPosition(){
		
	}
	
	// this is just to move a piece
	public void movePiece(String move){
		if( Integer.parseInt(move.substring(0,1)) > 7){ // this is the case if we have a pawn promotion
			chessboard[0][Integer.parseInt(move.substring(2,3))] = move.substring(3,4);
			chessboard[1][Integer.parseInt(move.substring(1,2))] = "e";
		} else {
			chessboard[Integer.parseInt(move.substring(2,3))][Integer.parseInt(move.substring(3,4))] = chessboard[Integer.parseInt(move.substring(0,1))][Integer.parseInt(move.substring(1,2))];
			chessboard[Integer.parseInt(move.substring(0,1))][Integer.parseInt(move.substring(1,2))] = "e";			
			if(chessboard[Integer.parseInt(move.substring(2,3))][Integer.parseInt(move.substring(3,4))].equals("A")){
				kingRowC = Integer.parseInt(move.substring(2,3));
				kingColC = Integer.parseInt(move.substring(3,4));
			}
		}
	}
	
	// undo a move
	public void unmovePiece(String move){
		if(Integer.parseInt(move.substring(0,1)) > 7){ // this is the case if we have a pawn promotion
			chessboard[0][Integer.parseInt(move.substring(2,3))] = move.substring(4,5);
			chessboard[1][Integer.parseInt(move.substring(1,2))] = "P";
		} else {
			chessboard[Integer.parseInt(move.substring(0,1))][Integer.parseInt(move.substring(1,2))] = chessboard[Integer.parseInt(move.substring(2,3))][Integer.parseInt(move.substring(3,4))];
			chessboard[Integer.parseInt(move.substring(2,3))][Integer.parseInt(move.substring(3,4))] = move.substring(4,5);
			if(chessboard[Integer.parseInt(move.substring(0,1))][Integer.parseInt(move.substring(1,2))].equals("A")){
				kingRowC = Integer.parseInt(move.substring(0,1));
				kingColC = Integer.parseInt(move.substring(1,2));
			}
		}
	}
	
	// this rotates the board...essentially it allows me to only consider moves for both
	// players while really only considering moves for "one" player. Imagine playing a
	// chess game by yourself, making a move, rotating the board 180 degrees, making a move,
	// and continuing in this manner
	// Using this method ended up vastly simplifying this program
	public void rotateBoard(){
		for(int i = 0; i < 4; i++){
			for(int j = 0; j < 8; j++){
				
				if(chessboard[i][j].equals("A")){
					kingRowL = 7-i;
					kingColL = 7-j;
				} else if (chessboard[i][j].equals("a")){
					kingRowC = 7-i;
					kingColC=7-j;
				}
				
				if(chessboard[7-i][7-j].equals("A")){
					kingRowL = i;
					kingColL = j;
				} else if (chessboard[7-i][7-j].equals("a")){
					kingRowC = i;
					kingColC = j;
				}
				
				String temp = chessboard[7-i][7-j];
				chessboard[7-i][7-j] = chessboard[i][j];
				chessboard[i][j] = temp;
				
				// for the spot located at i, j
				if( chessboard[i][j].equals("e") ){
					// do nothing to the "e"
				} else if ( Character.isUpperCase(chessboard[i][j].charAt(0)) ) {
					chessboard[i][j] = chessboard[i][j].toLowerCase(); // change the case of the upper case letter
				} else {
					chessboard[i][j] = chessboard[i][j].toUpperCase(); // change the case of the lower case letter
				}
				// for the spot we want to switch with i,j
				if( chessboard[7-i][7-j].equals("e") ){
					// do nothing to the "e"
				} else if ( Character.isUpperCase(chessboard[7-i][7-j].charAt(0)) ) {
					chessboard[7-i][7-j] = chessboard[7-i][7-j].toLowerCase(); // change the case of the upper case letter
				} else {
					chessboard[7-i][7-j] = chessboard[7-i][7-j].toUpperCase(); // change the case of the lower case letter
				}

			}
		}
	}
	
	// my move generation method
	public String possibleMoves(){
		String allpossiblemoves = "";
		for(int i = 0; i < chessboard.length; i++){
			for(int j = 0; j < chessboard.length; j++){
				if(chessboard[i][j].equals("R")){
					allpossiblemoves = allpossiblemoves + rookMoves(i,j);
				} else if(chessboard[i][j].equals("K")){
					allpossiblemoves = allpossiblemoves + knightMoves(i,j);
				} else if(chessboard[i][j].equals("B")){
					allpossiblemoves = allpossiblemoves + bishopMoves(i,j);
				} else if(chessboard[i][j].equals("Q")){
					allpossiblemoves = allpossiblemoves + queenMoves(i,j);
				} else if(chessboard[i][j].equals("A")){
					allpossiblemoves = allpossiblemoves + kingMoves(i,j);
				} else if(chessboard[i][j].equals("P")){
					allpossiblemoves = allpossiblemoves + pawnMoves(i,j);
				}
			}
		}
		
		return allpossiblemoves;
	}
	
	// most of the below methods are individual pieces' move generation method, although note
	// that kingSafety() is a very important method, and has a lot to do with why 
	public String rookMoves(int row, int col){
		String rookMoves = "";
		String removedPiece = "";
		int colMonitor = col;
		int rowMonitor = row;
		for(int i=-1 ; i<2 ; i++ ){
			for(int j=-1 ; j<2 ; j++){
				rowMonitor+=i;
				colMonitor+=j;
				if( (i==-1&&j==0)||(i==1&&j==0)||(i==0&&j==1)||(i==0&&j==-1) ){
					if (rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
						while(chessboard[rowMonitor][colMonitor].equals("e") && rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
							removedPiece = chessboard[rowMonitor][colMonitor];
							chessboard[rowMonitor][colMonitor] = "R";
							chessboard[row][col]="e";
							if(kingSafety()){
								rookMoves = rookMoves + createMove(row,col,rowMonitor,colMonitor,removedPiece);
							}
							chessboard[rowMonitor][colMonitor] = removedPiece;
							chessboard[row][col] = "R";
							rowMonitor+=i;
							colMonitor+=j;
							if(rowMonitor<0 || colMonitor<0 || rowMonitor>=8 || colMonitor>=8){
								break;
							}
						}
						if(rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
							if(Character.isLowerCase(chessboard[rowMonitor][colMonitor].charAt(0))){
								removedPiece = chessboard[rowMonitor][colMonitor];
								chessboard[rowMonitor][colMonitor] = "R";
								chessboard[row][col]="e";
								if(kingSafety()){
									rookMoves = rookMoves + createMove(row,col,rowMonitor,colMonitor,removedPiece);
								}
								chessboard[rowMonitor][colMonitor] = removedPiece;
								chessboard[row][col] = "R";
							}
						}
					}
				}
				rowMonitor=row;
				colMonitor=col;
			}
		}
		return rookMoves;
	}

	public String knightMoves(int row, int col){
		String knightMoves = "";
		String removedPiece = "";
		int colMonitor = col;
		int rowMonitor = row;
		
		for(int i=-2; i<3; i+=4){
			for(int j = -1; j<2 ; j+=2){
				rowMonitor+=i;
				colMonitor+=j;
				if(rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
					if( Character.isLowerCase(chessboard[rowMonitor][colMonitor].charAt(0)) ){
						removedPiece = chessboard[rowMonitor][colMonitor];
						chessboard[rowMonitor][colMonitor] = "K";
						chessboard[row][col]="e";
						if(kingSafety()){
							knightMoves = knightMoves + createMove(row,col,rowMonitor,colMonitor,removedPiece);
						}
						chessboard[rowMonitor][colMonitor] = removedPiece;
						chessboard[row][col] = "K";
					}
				}
				rowMonitor=row;
				colMonitor=col;
			}
		}
		
		for(int i=-1; i<2; i+=2){
			for(int j = -2; j<3 ; j+=4){
				rowMonitor+=i;
				colMonitor+=j;
				if(rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
					if( Character.isLowerCase(chessboard[rowMonitor][colMonitor].charAt(0)) ){
						removedPiece = chessboard[rowMonitor][colMonitor];
						chessboard[rowMonitor][colMonitor] = "K";
						chessboard[row][col]="e";
						if(kingSafety()){
							knightMoves = knightMoves + createMove(row,col,rowMonitor,colMonitor,removedPiece);
						}
						chessboard[rowMonitor][colMonitor] = removedPiece;
						chessboard[row][col] = "K";
					}
				}
				rowMonitor=row;
				colMonitor=col;
			}
		}
		
		return knightMoves;
	}
	
	public String bishopMoves(int row, int col){
		String bishopMoves = "";
		String removedPiece = "";
		int colMonitor = col;
		int rowMonitor = row;
		for(int i=-1 ; i<2 ; i+=2 ){
			for(int j=-1 ; j<2 ; j+=2){
				rowMonitor+=i;
				colMonitor+=j;
				if (rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
					while(chessboard[rowMonitor][colMonitor].equals("e") && rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
						removedPiece = chessboard[rowMonitor][colMonitor];
						chessboard[rowMonitor][colMonitor] = "B";
						chessboard[row][col]="e";
						if(kingSafety()){
							bishopMoves = bishopMoves + createMove(row,col,rowMonitor,colMonitor,removedPiece);
						}
						chessboard[rowMonitor][colMonitor] = removedPiece;
						chessboard[row][col] = "B";
						rowMonitor+=i;
						colMonitor+=j;
						if(rowMonitor<0 || colMonitor<0 || rowMonitor>=8 || colMonitor>=8){
							break;
						}
					}
					if(rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
						if(Character.isLowerCase(chessboard[rowMonitor][colMonitor].charAt(0))){
							removedPiece = chessboard[rowMonitor][colMonitor];
							chessboard[rowMonitor][colMonitor] = "B";
							chessboard[row][col]="e";
							if(kingSafety()){
								bishopMoves = bishopMoves + createMove(row,col,rowMonitor,colMonitor,removedPiece);
							}
							chessboard[rowMonitor][colMonitor] = removedPiece;
							chessboard[row][col] = "B";
						}
					}
				}
				rowMonitor=row;
				colMonitor=col;
			}
		}
		return bishopMoves;
	}
	
	public String queenMoves(int row, int col){
		String queenMoves = "";
		String removedPiece = "";
		int colMonitor = col;
		int rowMonitor = row;
		for(int i=-1 ; i<2 ; i++ ){
			for(int j=-1 ; j<2 ; j++){
				rowMonitor+=i;
				colMonitor+=j;
				if (rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
					while(chessboard[rowMonitor][colMonitor].equals("e") && rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
						removedPiece = chessboard[rowMonitor][colMonitor];
						chessboard[rowMonitor][colMonitor] = "Q";
						chessboard[row][col]="e";
						if(kingSafety()){
							queenMoves = queenMoves + createMove(row,col,rowMonitor,colMonitor,removedPiece);
						}
						chessboard[rowMonitor][colMonitor] = removedPiece;
						chessboard[row][col] = "Q";
						rowMonitor+=i;
						colMonitor+=j;
						if(rowMonitor<0 || colMonitor<0 || rowMonitor>=8 || colMonitor>=8){
							break;
						}
					}
					if(rowMonitor>=0 && colMonitor>=0 && rowMonitor<8 && colMonitor<8){
						if(Character.isLowerCase(chessboard[rowMonitor][colMonitor].charAt(0))){
							removedPiece = chessboard[rowMonitor][colMonitor];
							chessboard[rowMonitor][colMonitor] = "Q";
							chessboard[row][col]="e";
							if(kingSafety()){
								queenMoves = queenMoves + createMove(row,col,rowMonitor,colMonitor,removedPiece);
							}
							chessboard[rowMonitor][colMonitor] = removedPiece;
							chessboard[row][col] = "Q";
						}
					}
				}
				rowMonitor=row;
				colMonitor=col;
			}
		}
		return queenMoves;
	}
	
	public String kingMoves(int row, int col){
		String kingMoves = "";
		String removedPiece = "";
		int colMonitor = col;
		int rowMonitor = row;
		if(col>0){
			colMonitor--;
		}
		if (row>0){
			rowMonitor--;
		}
		int restoreCol=colMonitor;
		
		for(int i = 0; i < 3 && rowMonitor<8;i++){
			for(int j = 0; j < 3 && colMonitor<8;j++){
				if(rowMonitor==row && colMonitor==col){
					colMonitor++;// this makes it so that we don't create move where king moves to spot its in
				}else if(Character.isLowerCase(chessboard[rowMonitor][colMonitor].charAt(0))){
					removedPiece = chessboard[rowMonitor][colMonitor];
					chessboard[rowMonitor][colMonitor]="A";
					chessboard[row][col]="e";
					kingRowC = rowMonitor;
					kingColC = colMonitor;
					if(kingSafety()){
						kingMoves = kingMoves + createMove(row,col,rowMonitor,colMonitor, removedPiece);
					}
					chessboard[row][col]="A";
					chessboard[rowMonitor][colMonitor]=removedPiece;
					kingRowC = row;
					kingColC = col;

					colMonitor++;
				} else {
					colMonitor++;
				}
			}
			colMonitor = restoreCol;
			rowMonitor++;
		}

		return kingMoves;
	}
	
	public String pawnMoves(int row, int col){
		//pawn capture
		String pawnMoves = "";
		if( (row-1)>=1 ){
			for( int i = -1; i < 2; i+=2){
				try{	
					if( Character.isLowerCase(chessboard[row-1][col+i].charAt(0)) && !chessboard[row-1][col+i].equals("e") ){
						String removedPiece = chessboard[row-1][col+i];
						chessboard[row-1][col+i] = "P";
						chessboard[row][col]="e";
						if(kingSafety()){
							pawnMoves = pawnMoves + createMove(row,col,row-1,col+i,removedPiece);
						}
						chessboard[row-1][col+i] = removedPiece;
						chessboard[row][col] = "P";
					}
				} catch (Exception e){}
			}
		}
		
		//pawn promotion and capture
		if( (row-1)==0 ){
			for( int i = -1 ; i < 2 ; i+=2){
				try{
					if( Character.isLowerCase(chessboard[row-1][col+i].charAt(0)) && !chessboard[row-1][col+i].equals("e") ){
						String [] promotionPieces = {"Q","K","B","R"};
						for(int j = 0 ; j < 4 ; j++){
							String removedPiece = chessboard[row-1][col+i];
							chessboard[row-1][col+i] = promotionPieces[j];
							chessboard[row][col]="e";
							if(kingSafety()){
								pawnMoves = pawnMoves + "8" + col + (col+i)+promotionPieces[j]+removedPiece;
							}
							chessboard[row-1][col+i] = removedPiece;
							chessboard[row][col] = "P";
						}
					}
				} catch (Exception e) {}
			}
		}
		
		// move one up
		if( (row-1)>=1 ){
			try{
				if( chessboard[row-1][col].equals("e") ){
					String removedPiece = chessboard[row-1][col];
					chessboard[row-1][col] = "P";
					chessboard[row][col]="e";
					if(kingSafety()){
						pawnMoves = pawnMoves + createMove(row,col,row-1,col,removedPiece);
					}
					chessboard[row-1][col] = removedPiece;
					chessboard[row][col] = "P";
				}
			} catch (Exception e) {}
		}
		
		// move one up and be promoted
		if(row==1){
			try{
				if( chessboard[row-1][col].equals("e") ){
					String [] promotionPieces = {"Q","K","B","R"};
					for(int j = 0 ; j < 4 ; j++){
						String removedPiece = chessboard[row-1][col];
						chessboard[row-1][col] = promotionPieces[j];
						chessboard[row][col]="e";
						if(kingSafety()){
							pawnMoves = pawnMoves + "8" + col + (col)+promotionPieces[j]+removedPiece;
						}
						chessboard[row-1][col] = removedPiece;
						chessboard[row][col] = "P";
					}
				}
			} catch (Exception e) {}
		}
		
		// move two up from starting position
		if( row==6 ){
			try{
				if( chessboard[row-1][col].equals("e") && chessboard[row-2][col].equals("e") ){
					String removedPiece = chessboard[row-2][col];
					chessboard[row-2][col] = "P";
					chessboard[row][col]="e";
					if(kingSafety()){
						pawnMoves = pawnMoves + createMove(row,col,row-2,col,removedPiece);
					}
					chessboard[row-2][col] = removedPiece;
					chessboard[row][col] = "P";
				}
			} catch (Exception e) {}
		}	
		
		return pawnMoves;
	}
	
	public String createMove(int r, int c, int currentrow, int currentcol, String removedPiece){
		String move = Integer.toString(r) + Integer.toString(c) + Integer.toString(currentrow) + Integer.toString(currentcol) + removedPiece;
		return move;
	}
	
	public boolean kingSafety(){
		//check for diagonal attacks from the bishop and the queen
		int temprow = kingRowC;
		int tempcol = kingColC;
		for(int i = -1 ; i < 2 ; i+=2){
			for(int j = -1 ; j < 2 ; j += 2){
				temprow+=i;
				tempcol+=j;
				if(temprow>=0 && tempcol>=0 && temprow < 8 && tempcol < 8){
					while(chessboard[temprow][tempcol].equals("e")){
						temprow+=i;
						tempcol+=j;
						if(temprow<0 || tempcol<0 || temprow > 7 || tempcol > 7){
							break;
						}
					}
					if(temprow>=0 && tempcol>=0 && temprow < 8 && tempcol < 8){
						if(chessboard[temprow][tempcol].equals("b") || chessboard[temprow][tempcol].equals("q")){
							return false;
						}
					}
				}
				temprow=kingRowC;
				tempcol=kingColC;
			}
		}
		
		
		//check for horizontal and vertical attacks from the rook and queen
		for(int i = -1; i < 2; i++){
			if(i==0){
				for(int j = -1; j < 2; j+=2){
					tempcol+=j;
					if(tempcol>=0 && tempcol<8){
						while(chessboard[temprow][tempcol].equals("e")){
							tempcol+=j;
							if(tempcol>7 || tempcol < 0){
								break;
							}
						}
						if(tempcol>=0 && tempcol<8){
							if(chessboard[temprow][tempcol].equals("r") || chessboard[temprow][tempcol].equals("q")){
								return false;
							}
						}
					}
					tempcol = kingColC;
				}
			} else {
				temprow+=i;
				if(temprow>=0 && temprow<8){
					while(chessboard[temprow][tempcol].equals("e")){
						temprow+=i;
						if(temprow>7 || temprow < 0){
							break;
						}
					}
					if(temprow>=0 && temprow<8){
						if(chessboard[temprow][tempcol].equals("r") || chessboard[temprow][tempcol].equals("q")){
							return false;
						}
					}
				}
				temprow = kingRowC;
			}	
		}
		
		//check for attacks from a knight
		for(int i=-2; i<3; i+=4){
			for(int j = -1; j<2 ; j+=2){
				temprow+=i;
				tempcol+=j;
				if(temprow>=0 && tempcol>=0 && temprow<8 && tempcol<8){
					if( chessboard[temprow][tempcol].equals("k") ){
						return false;
					}
				}
				temprow=kingRowC;
				tempcol=kingColC;
			}
		}
		
		for(int i=-1; i<2; i+=2){
			for(int j = -2; j<3 ; j+=4){
				temprow+=i;
				tempcol+=j;
				if(temprow>=0 && tempcol>=0 && temprow<8 && tempcol<8){
					if( chessboard[temprow][tempcol].equals("k") ){
						return false;
					}
				}
				temprow=kingRowC;
				tempcol=kingColC;
			}
		}
		
		//check for attacks from a pawn
		for(int i = -1; i < 2 ; i+=2){
			temprow+=-1;
			tempcol+=i;
			if(temprow>=0 && tempcol>=0 && temprow<7 && tempcol<8){//the reason that I have temprow<7 here is because if temprow is 7 then the pawn is going to be promoted, so it'll work a bit differently
				if( chessboard[temprow][tempcol].equals("p") ){
					return false;
				}
			}
			temprow=kingRowC;
			tempcol=kingColC;
		}
		
		//check for attacks from a king
		int rowcounter = 0;
		int colcounter = 0;
		if(temprow-1>=0){
			temprow--;
		}else{
			rowcounter++;
		}
		if(tempcol-1>=0){
			tempcol--;
		}else{
			colcounter++;
		}
		int restoreCol = tempcol;
		for(int row = rowcounter ; row<3 && temprow<8; row++){
			for( int col = colcounter; col<3 && tempcol<8 ; col++){
				if(kingRowC==4 && kingColC==2){

				}
				if( chessboard[temprow][tempcol].equals("a") ){
					return false;
				}
				tempcol++;
			}
			tempcol = restoreCol;
			temprow++;
		}
		
		
		return true;
	}
	
	// the arrays below are positional value tables, which are commonly used in chess
	// engines to help evaluate whether or not a piece is in a "good" position
	// I found out about these on: 
	// https://chessprogramming.wikispaces.com/Simplified+evaluation+function
	// Basically, what these do is discourage or encourage different moves
	// of different pieces at various points in the game
	int pawns[][] = {
		{0, 0, 0, 0, 0, 0, 0, 0},
		{50, 50, 50, 50, 50, 50, 50, 50},
		{10, 10, 20, 30, 30, 20, 10, 10},
		{5, 5, 10, 25, 25, 10, 5, 5},
		{0, 0, 0, 50, 50, 0, 0, 0},
		{5, -5, -10, 0, 0, -10, -5, 5},
		{5, 10, 10,-20,-20, 10, 10, 5},// white player: 10, black player: 30
		{0, 0, 0, 0, 0, 0, 0, 0}
	};

	int knights [][] = {
		{-50,-40,-30,-30,-30,-30,-40,-50},
		{-40,-20,  0,  0,  0,  0,-20,-40},
		{-30,  0, 10, 15, 15, 10, 0,-30},
		{-30,  5, 15, 20, 20, 15,  5,-30},
		{-30,  0, 15, 20, 20, 15,  0,-30},
		{-30,  5, 50, 15, 15, 50,  5,-30},
		{-40,-20,  0,  5,  5,  0,-20,-40}, //white player: -30, black player: -80
		{-50,-40,-30,-30,-30,-30,-40,-50}

	};


	int bishops [][] = {
		{-20,-10,-10,-10,-10,-10,-10,-20},
		{-10,  0,  0,  0,  0,  0,  0,-10},
		{-10,  0,  5, 10, 10,  5,  0,-10},
		{-10,  5,  5, 10, 10,  5,  5,-10},
		{-10,  0, 10, 10, 10, 10,  0,-10},
		{-10, 10, 10, 10, 10, 10, 10,-10},
		{-10,  5,  0,  0,  0,  0,  5,-10},//both: -20
		{-20,-10,-10,-10,-10,-10,-10,-20}
	};
	 
	 int rooks [][] = {
		{0, 0, 0, 0, 0, 0, 0, 0},	 
		{5, 10, 10, 10, 10, 10, 10, 5},
		{-5, 0, 0, 0, 0, 0, 0, -5},
		{-5, 0, 0, 0, 0, 0, 0, -5},
		{-5, 0, 0, 0, 0, 0, 0, -5},
		{-5, 0, 0, 0, 0, 0, 0, -5},//both: 0
		{-5, 0, 0, 0, 0, 0, 0, -5},
		{0, 0, 0, 5, 5, 0, 0, 0}
	  };
	  
	  
	  
	  int queen [][] = {
		{-20,-10,-10, -5, -5,-10,-10,-20},
		{-10, 0, 0, 0, 0, 0, 0,-10},
		{-10, 0, 5, 5, 5, 5, 0,-10},
		{-5, 0, 5, 5, 5, 5, 0, -5},//both: -5
		{0, 0, 5, 5, 5, 5, 0, -5},
		{-10, 5, 5, 5, 5, 5, 0,-10},
		{-10, 0, 5, 0, 0, 0, 0,-10},
		{-20,-10,-10, -5, -5,-10,-10,-20}
	  };
	  
	  
	  
	  int midgameking[][] = { 
		{-30,-40,-40,-50,-50,-40,-40,-30},
		{-30,-40,-40,-50,-50,-40,-40,-30},
		{-30,-40,-40,-50,-50,-40,-40,-30},
		{-30,-40,-40,-50,-50,-40,-40,-30},
		{-20,-30,-30,-40,-40,-30,-30,-20},// both:0
		{-10,-20,-20,-20,-20,-20,-20,-10},
		{20, 20,  0,  0,  0,  0, 20, 20},
		{20, 30, 10,  0,  0, 10, 30, 20}
	  };
	   
	   
	  int endgameking[][] = {
		{-50,-40,-30,-20,-20,-30,-40,-50},
		{-30,-20,-10,  0,  0,-10,-20,-30},
		{-30,-10, 20, 30, 30, 20,-10,-30},
		{-30,-10, 30, 40, 40, 30,-10,-30},
		{-30,-10, 30, 40, 40, 30,-10,-30},
		{-30,-10, 20, 30, 30, 20,-10,-30},
		{-30,-30,  0,  0,  0,  0,-30,-30},
		{-50,-30,-30,-30,-30,-30,-30,-50}
	  };
		  
		  public int heuristic(int numPossMoves, int depth){
			  //by having the number of possibleMoves, we will know if in checkmate or stalemate
			  //by having the depth we can weight the scores according to how far in the tree we are
			  
			  // evaluate board from white point of view
			  int evaluationScore = 0, gamePhaseBasedOnPieces = evaluateBoardMaterial();
			  evaluationScore += gamePhaseBasedOnPieces;
			  evaluationScore += evaluateBoardPosition(gamePhaseBasedOnPieces);
			  evaluationScore += evaluateMoveability(numPossMoves, depth, gamePhaseBasedOnPieces);
			  evaluationScore += evaluateAttacks();
			  
			  // evaluate board from black point of view
			  rotateBoard();
			  gamePhaseBasedOnPieces = evaluateBoardMaterial();
			  evaluationScore -= gamePhaseBasedOnPieces;
			  evaluationScore -= evaluateBoardPosition(gamePhaseBasedOnPieces);
			  evaluationScore -= evaluateMoveability(numPossMoves, depth, gamePhaseBasedOnPieces);
			  evaluationScore -= evaluateAttacks();
			  
			  rotateBoard();
			  return -(evaluationScore+depth*40);
		  }
		  
		  public int evaluateBoardMaterial(){
			int materialScore = 0;
			for(int i = 0; i < chessboard.length; i++){
				for(int j = 0; j < chessboard.length; j++){				
					if(chessboard[i][j].equals("A")){
						materialScore+=kingCapValue;
					}else if(chessboard[i][j].equals("K")){
						materialScore+=knightCapValue;
					}else if(chessboard[i][j].equals("B")){
						materialScore+=bishopCapValue;
					}else if(chessboard[i][j].equals("Q")){
						materialScore+=queenCapValue;
					}else if(chessboard[i][j].equals("R")){
						materialScore+=rookCapValue;
					}else if(chessboard[i][j].equals("P")){
						materialScore+=pawnCapValue;
					}
				}
			}
			return materialScore;
		  }
		  
		  // use the PVT's to evaluate your board position
		  public int evaluateBoardPosition(int gamePhaseBasedOnPieces){
				int positionalScore = 0;
			  for(int i = 0; i < chessboard.length; i++){
					for(int j = 0; j < chessboard.length; j++){				
						if(chessboard[i][j].equals("A")){
							if(gamePhaseBasedOnPieces > 2000){
								positionalScore += midgameking[i][j];
							} else {
								positionalScore += endgameking[i][j];
							}
						}else if(chessboard[i][j].equals("K")){
							positionalScore+=knights[i][j];
						}else if(chessboard[i][j].equals("B")){
							positionalScore+=bishops[i][j];
						}else if(chessboard[i][j].equals("Q")){
							positionalScore+=queen[i][j];
						}else if(chessboard[i][j].equals("R")){
							positionalScore+=rooks[i][j];
						}else if(chessboard[i][j].equals("P")){
							positionalScore+=pawns[i][j];
						}
					}
				}
			  return positionalScore;
		  }
		  
		  
		  // check for checkmate and stalemate
		  public int evaluateMoveability(int numPossMoves, int depth, int gamePhaseBasedOnPieces){
			  
			  int moveabilityScore = 0;
			  
			  if(numPossMoves == 0){
				  //checkmate
				  if(!kingSafety()){
					  moveabilityScore-=50000*depth;
				// stalemate
				  } else {
					  moveabilityScore-=25000*depth;
				  }
			  }
			  
			  return moveabilityScore;
		  }
		  

		  // this evaluates how many of your pieces are being attacked
		  public int evaluateAttacks(){
			  
			  int attackScore = 0;
			  
			  int tempKingRow = kingRowC;
			  int tempKingCol = kingColC;
			  
			  for(int i = 0; i < chessboard.length; i++){
					for(int j = 0; j < chessboard.length; j++){				
						if(chessboard[i][j].equals("K")){
							kingRowC = i;
							kingColC = j;
							if(!kingSafety()){
								attackScore-=150;
							}
						}else if(chessboard[i][j].equals("B")){
							kingRowC = i;
							kingColC = j;
							if(!kingSafety()){
								attackScore-=150;
							}
						}else if(chessboard[i][j].equals("Q")){
							kingRowC = i;
							kingColC = j;
							if(!kingSafety()){
								attackScore-=450;
							}
						}else if(chessboard[i][j].equals("R")){
							kingRowC = i;
							kingColC = j;
							if(!kingSafety()){
								attackScore-=250;
							}
						}else if(chessboard[i][j].equals("P")){
							kingRowC = i;
							kingColC = j;
							if(!kingSafety()){
								attackScore-=50;
							}
						}
					}
				}
			  kingRowC = tempKingRow;
			  kingColC = tempKingCol;
			  
			  return attackScore;
		  }
}
