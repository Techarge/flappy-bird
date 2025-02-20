<!-- src/routes/+page.svelte -->
<script lang="ts">
    import { onMount } from 'svelte';

    // Game constants
    const GRAVITY = 0.25;
    const JUMP = -6;
    const PIPE_SPEED = 2;
    const PIPE_SPACING = 200;
    const GAP_HEIGHT = 150;

    // Game state
    let birdY = 250;
    let birdVelocity = 0;
    let pipes: { x: number; topHeight: number }[] = [];
    let score = 0;
    let gameOver = false;
    let gameStarted = false;

    // Canvas reference
    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;

    // Start or restart game
    function startGame() {
        gameStarted = true;
        gameOver = false;
        birdY = 250;
        birdVelocity = 0;
        pipes = [{ x: 600, topHeight: getRandomHeight() }];
        score = 0;
    }

    // Handle jump
    function jump() {
        if (!gameStarted) startGame();
        if (!gameOver) birdVelocity = JUMP;
    }

    // Generate random pipe height
    function getRandomHeight() {
        return Math.random() * 200 + 50;
    }

    // Game loop
    function update() {
        if (!gameStarted || gameOver) return;

        // Update bird
        birdVelocity += GRAVITY;
        birdY += birdVelocity;

        // Update pipes
        pipes = pipes.map(pipe => ({ ...pipe, x: pipe.x - PIPE_SPEED }));
        if (pipes[0]?.x < -50) pipes.shift();
        if (pipes[pipes.length - 1].x < 400) {
            pipes = [...pipes, { x: 600, topHeight: getRandomHeight() }];
        }

        // Collision detection
        pipes.forEach(pipe => {
            if (
                (pipe.x < 50 && pipe.x > 0) &&
                (birdY < pipe.topHeight || birdY > pipe.topHeight + GAP_HEIGHT)
            ) {
                gameOver = true;
            }
        });

        // Ground/Ceiling collision
        if (birdY > 500 || birdY < 0) gameOver = true;

        // Update score
        pipes.forEach(pipe => {
            if (pipe.x + 50 < 50 && pipe.x + 50 > 48) score += 1;
        });
    }

    // Render game
    function draw() {
        if (!ctx) return;

        ctx.clearRect(0, 0, 600, 500);

        // Draw bird
        ctx.fillStyle = 'yellow';
        ctx.beginPath();
        ctx.arc(50, birdY, 20, 0, Math.PI * 2);
        ctx.fill();

        // Draw pipes
        ctx.fillStyle = 'green';
        pipes.forEach(pipe => {
            ctx.fillRect(pipe.x, 0, 50, pipe.topHeight);
            ctx.fillRect(pipe.x, pipe.topHeight + GAP_HEIGHT, 50, 500);
        });

        // Draw score
        ctx.fillStyle = 'black';
        ctx.font = '20px Arial';
        ctx.fillText(`Score: ${score}`, 10, 30);

        if (gameOver) {
            ctx.fillStyle = 'red';
            ctx.font = '40px Arial';
            ctx.fillText('Game Over', 200, 250);
        }
    }

    onMount(() => {
        ctx = canvas.getContext('2d')!;
        
        // Game loop
        const gameLoop = () => {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        };
        gameLoop();

        // Event listeners
        window.addEventListener('keydown', (e) => {
            if (e.code === 'Space') jump();
        });
        canvas.addEventListener('click', jump);
    });
</script>

<div class="flex flex-col items-center justify-center min-h-screen bg-blue-200">
    <h1 class="text-4xl font-bold mb-4">Flappy Bird</h1>
    <canvas
        bind:this={canvas}
        width="600"
        height="500"
        class="border-2 border-black bg-white"
    ></canvas>
    {#if !gameStarted}
        <p class="mt-4 text-lg">Click or press Space to start!</p>
    {/if}
    {#if gameOver}
        <div class="mt-4 flex flex-col items-center">
            <p class="text-xl mb-2">Final Score: {score}</p>
            <button
                on:click={startGame}
                class="px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600 transition"
            >
                Restart Game
            </button>
        </div>
    {/if}
</div>
