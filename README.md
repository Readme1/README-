README-/README.md
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Readme</title>
  
<style type="text/css">
</style>
</head>
<body onload="tetris.play('scene')" onkeydown="tetris.handleKey(event)">

<script src="jquery.js"></script>
<script>
score = 0;
tetris = {
		types : [[';', '(', ')', '{', '}'],
		         ['=', '!=', '==', '<', '>'],
		         ['1', '2', '4', '8'],
		         ['+', '*', '/'],
		         [' '],
		         ['score', 'a', 'b', 'c', 'x', 'y', 'z'],
		         ['for', 'if', 'while', 'do', 'function', 'return']
		         ],
		         
		sizes : [1, 1, 1, 1, 1, 2, 1],
		colors : ['#900', '#700', '#800', '#990', '#099', '#909', '#444'],
		bgcolors : ['#fcc', '#cfc', '#ccf', '#ffc', '#cff', '#fcf', '#ccc'],
		         
		blocks : [],
		
		margin : 5,
		h : 16, // in blockw
		w : 16, // in blockw
		blockw : 30,
		
		play : function(canvasId) {
			var scene = document.getElementById(canvasId);
			tetris.ctx = scene.getContext('2d');
			
			tetris.nextBlock = tetris.newBlock();
			tetris.loop();
		},
		
		drawBoard : function() {
			var ctx = tetris.ctx; 
			ctx.lineWidth = 2;
			
			ctx.clearRect(0, 0, 700, 700);
			
			ctx.fillStyle = "rgb(200,0,0)";
			ctx.fillText("Score: "+score, tetris.w*tetris.blockw + 20, 30);
			
			this.ctx.strokeStyle = "rgb(200,0,0)";
			this.ctx.fillStyle = "rgb(255,255,255)";
			
			ctx.beginPath();
			tetris.rect(0,0,tetris.w*tetris.blockw,tetris.h*tetris.blockw);
			ctx.fill();
			ctx.stroke();
		},
		
		rect : function(x, y, width, height, radius) {
			radius = radius || 5;
			x += tetris.margin;
			y += tetris.margin;
			var ctx = tetris.ctx;
			ctx.moveTo(x + radius, y);
			ctx.lineTo(x + width - radius, y);
			ctx.quadraticCurveTo(x + width, y, x + width, y + radius);
			ctx.lineTo(x + width, y + height - radius);
			ctx.quadraticCurveTo(x + width, y + height, x + width - radius, y + height);
			ctx.lineTo(x + radius, y + height);
			ctx.quadraticCurveTo(x, y + height, x, y + height - radius);
			ctx.lineTo(x, y + radius);
			ctx.quadraticCurveTo(x, y, x + radius, y);
		},
		
		drawBlocks : function() {
			if (tetris.nextBlock) {
				tetris.drawBlock(tetris.nextBlock, (tetris.w-tetris.nextBlock.x)*tetris.blockw + 20, 80);
			}
			
			if (tetris.block) {
				tetris.drawBlock(tetris.block);
			}
			for (var i in tetris.blocks) {
				var b = tetris.blocks[i];
				tetris.drawBlock(b);
			}
		},
		
		drawBlock : function(b,x,y) {
			x = x || 0;
		 	y = y || 0;
		 	var w = tetris.sizes[b.type];
			var ctx = tetris.ctx; 
			ctx.beginPath();
			tetris.rect(x+b.x*tetris.blockw,y+b.y*tetris.blockw, w*tetris.blockw,tetris.blockw);
			ctx.strokeStyle = tetris.colors[b.type];
			ctx.fillStyle = tetris.bgcolors[b.type];
			ctx.fill();
			ctx.stroke();
			ctx.fillStyle = tetris.colors[b.type];
			var txt = tetris.types[b.type][b.seq];
			ctx.fillText(txt, x+(b.x+0.35)*tetris.blockw,y+(b.y+0.65)*tetris.blockw);
		},
		
		setGameOver : function() {
			tetris.gameOver = true;
			
			var ctx = tetris.ctx;
			ctx.fillStyle = "rgb(0,0,0)";
			ctx.fillText("GAME OVER", 100, 100);
		},
		
		checkSuccess : function(row) {
			var rowBlocks = [];
			
			for (var i in tetris.blocks) {
				var b = tetris.blocks[i];
				if (b.y === row) {
					rowBlocks.push(b);
				}
			}
			
			var success = tetris.isRowComplete(rowBlocks);
			
			if (!success) {
				return;
			}
			
			score += 100;
			
			// move all one row down and remove scored row
			for (var i = tetris.blocks.length -1; i >= 0; i--) {
				var b = tetris.blocks[i];
				if (b.y === row) {
					tetris.blocks.splice(i, 1);
				} else if (b.y < row) {
					b.y++;
				}
			}
			
			rowBlocks.sort(function(a,b) {
				return a.x - b.x;
			});
			var str = "";
			for (var i in rowBlocks) {
				var b = rowBlocks[i];
				str += tetris.types[b.type][b.seq];
			}
			$("#out").append(str+"\n");
			try {
				eval(str);
			} catch(err) {
				$("#out").append(err);
				tetris.setGameOver();
			}
		},
		
		isRowComplete :function(blocks) {
			var len = 0;
			for (var i in blocks) {
				var b = blocks[i];
				len += tetris.sizes[b.type];
			}
			
			return len == tetris.w;
		},
		
		updateBlock : function() {
			if (tetris.block === undefined) {
				tetris.block = tetris.nextBlock;
				tetris.nextBlock = tetris.newBlock();
				return;
			}
			
			if (tetris.block.y+1 == tetris.h) {
				tetris.blocks.push(tetris.block);
				tetris.checkSuccess(tetris.block.y);
				
				var b = tetris.getBlock(tetris.nextBlock.x, tetris.nextBlock.y, tetris.sizes[tetris.newBlock.type]);
				if (b !== undefined) {
					tetris.setGameOver();
					return;
				}
				tetris.block = tetris.nextBlock;
				tetris.nextBlock = tetris.newBlock();
				return;
			}
			
			if (tetris.getBlock(tetris.block.x, tetris.block.y+1, tetris.sizes[tetris.block.type]) == undefined) {
				tetris.block.y += 1;
				return;
			}
			
			tetris.blocks.push(tetris.block);
			tetris.checkSuccess(tetris.block.y);
			
			var b = tetris.getBlock(tetris.nextBlock.x, tetris.nextBlock.y, tetris.sizes[tetris.block.type]);
			if (b !== undefined) {
				tetris.setGameOver();
				return;
			}
			tetris.block = tetris.nextBlock;
			tetris.nextBlock = tetris.newBlock();
		},
		
		getBlock : function (x, y, w) {
			w = w || 1;
			for (var i in tetris.blocks) {
				var b = tetris.blocks[i];
				if (((b.x < x + w) && (x < b.x + tetris.sizes[b.type])) 
						&& (b.y === y)) {
					return b;
				}
			}
			
			return undefined;
		},
		
		getTopBlock : function (x, w) {
			w = w || 1;
			var max = tetris.h + 1;
			var found = undefined;
			for (var i in tetris.blocks) {
				var b = tetris.blocks[i];
				if (((b.x < x + w) && (x < b.x + tetris.sizes[b.type])) && (b.y < max)) {
					max = b.y;
					found = b;
				}
			}
			
			return found;
		},
		
		newBlock : function() {
			var t = Math.round(Math.random() * 1000) % tetris.types.length;
			return {x : tetris.w/2 - 1, y : 0, type : t, seq : 0 };
		},
		
		redraw : function() {
			tetris.drawBoard();
			tetris.drawBlocks();
		},
		
		loop : function() {
			tetris.redraw();
			tetris.updateBlock();
			
			if (tetris.gameOver === undefined) {
				setTimeout(tetris.loop, 500);
			}
		},
		
		handleKey : function(event) {
			if (tetris.gameOver !== undefined) {
				return;
			}
			
			if (event.keyCode === 37) { // left
				if (tetris.block.x > 0) {
					tetris.block.x--;
				}
			} else if (event.keyCode === 39) { // right
				if (tetris.block.x + tetris.sizes[tetris.block.type] < tetris.w) {
					tetris.block.x++;
				}
			} else if (event.keyCode === 40) { // down
				var b = tetris.getTopBlock(tetris.block.x, tetris.sizes[tetris.block.type]);
			    if (!b) {
			    	tetris.block.y = tetris.h - 1;
			    } else {
			    	tetris.block.y = b.y - 1;
			    }
			} else if (event.keyCode === 38) { // up
					tetris.block.seq = (tetris.block.seq + 1) % (tetris.types[tetris.block.type].length);
			}
			
			tetris.redraw();
		}
}
</script>



<canvas id="scene" width="700" height="630">
Your browser is too old to play. Please do update.
</canvas>
<pre>
<div id="out">
</div>
</pre>
</body>
</html>
