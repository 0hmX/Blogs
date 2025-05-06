Target Audience: People who do not know how to make an emulator

Before this video start I want to let you guys know who is this video for,

This video is for people who knows what an emulator! have use some emulator to play games and shit.

But do not understand how to make one.

This is not a full fleged tutorial as I will be givig code but a lot of the implimenation will be missing

this is because I want you guys to do your own research and have fun! and totaly not because I am lazy!

# What is an emulator?

Now, I know you guys already know this, but for the sake of the video let's understand what an emulator is.

Here is my definition: emulators are software that mimic the hardware of another computer!

I focus on hardware because that is what we are going to focus on today.

We are going to create a CHIP-8 emulator.

# What is CHIP-8?

It's a virtual machine.

So unlike other systems that have actual hardware, it's a bunch of specifications made by some nerds which is agreed upon universally and is named CHIP-8.

The reason we will focus on CHIP-8 is because unlike something like a GameBoy it's much simpler.

A GameBoy emulator has at least 4 to 5 parts to it: PPU (basically GPU), APU for audio processing, Memory bus controller and CPU.

Our CHIP-8 has a CPU with an eight-bit array as memory and a single boolean for audio.

There is also thea a big diffrence in the number instructions 

where CHIP-8 has 35 instructions only while GameBoy has over 500.

# Architecture and coding?

For this part we will understand and code in parallel.

I will be using TypeScript for this project.

Let's use Vite, run `pnpm create vite@latest` and select vanilla TS project.

Let's start by defining our object called chip8

```ts
const chip8 = {

}
```

This will have a memory which is a `Uint8Array` of length 4096

### Why exactly 4096?

Before I say this, any number which has a 0x before it is a hex value in this video.

A hex value is base 16 representation of numbers. 

We use base 10, 0 to 9. 

In hex we go from 0 - F, F in decimal is 15 so 16 total characters!

Hex is a popular choice for writing memory addresses so be aware of it!

Coming back to why 4096?

The address range in CHIP-8 goes from 0x0 to 0x1000

0x1000 in decimal is 1 * 16^3 = 4096

```ts
const chip8 = {
    memory: new Uint8Array(4096)
}
```

## Let's talk about the CPU?

This is an over-simplification but any CPU, Intel i5-13540H or 4050 or SHARP Z80, all are a collection of:

Registers, these are little boxes present inside the CPU to store values for calculation, going from 0-F in CHIP-8.

And stack is an 16 length array that will store memory addresses in case of CHIP-8.

```ts
const cpu = {
    register: Array.from({ length: 0xF }, () => 0),
    stack: Array.from({ length: 16 }, () => 0),
    sp: 0,
    pc: 0,
    i: 0
}
```

## Let's talk about input?

The input in CHIP-8 is done using a hex keyboard; meaning the keycodes go from 0 to F.

The direction keys in CHIP-8 are '8', '4', '6' & '2'

We will store the input using a 16 length boolean array:

```ts
const input = Array.from({ length: 16 }, () => false)
```

when a key is pressed we will set the resective address in the array to true

## Finally we have the clock!

CHIP-8 has two types of clock:

1. Delay timer; used for timing events in games
2. Sound timer: used to make beeping sounds

Let's implement these timers:

```ts
const clock = {
    delayTimer: 0,
    soundTimer: 0
}
```

## Display

The chip8 draws using sprits it has a fixed width of 8 pixel and a variable height of 1 - F or 15 pixles in height. The sprite data is preserted as a unsiged 8bit number going form 0 to 255.

Lets say we want to create a sprite that will display

### collsision 

the drawin of sprits in chip8 is also used to detect collsion; we use the xor operator meaning

1(previous pixel data) xor 1(new pixel data to set) = 1

we set the carray flag VF register to 1 to indicate collsion!

let look at the implimentaion

```ts
const canvas = document.createElement('canvas');
canvas.width = 64;
canvas.height = 32;
document.body.appendChild(canvas);

const ctx = canvas.getContext('2d');

function updateDisplay(display) {
    for (let i = 0; i < display.length; i++) {
        ctx.fillStyle = display[i] ? '#0F0' : '#000';
        ctx.fillRect(i % 64, Math.floor(i / 64), 1, 1);
    }
}
```

[code explain]

we will create an canvas element and get a 2d context

finally we will loop over our diaply array whidh is an array of bool

if any value is set we will draw a green box of 1by1 pixel in length

[end]

the darw back of such a solution for collsion detection is the draw calls can not be merge together! 

order of drawing matters

## Storing and Reading Instructions

In Chip-8, programs are stored starting at memory address 0x200. The memory addresses from 0x000 to 0x1FF store the Chip-8 interpreter itself.

Chip-8 instructions are 16 bits long.

Remember that we have stored the instructions as 8-bit values.

Meaning we will need to read two at a time to get the full instruction.

[code_explain]

Let's start by making a `runCycle` function.

For this we will use a for loop because for each frame we need to read the instruction at least 10 times.

Then we will use the program counter to extract the instruction and send it to an `executeInstruction` function.

Let's look at the `executeInstruction` function.

This will take an instruction and match it with a particular condition.

Here the implementation is very basic, but in real life you have about 35 instructions to match.

I have kept it small as this is just an example.

[end]

```ts
function runCycle() {
    for (let i = 0; i < chip8.speed; i++) {
        if (!chip8.paused) {
            const opcode = (chip8.memory[chip8.cpu.pc] << 8) | chip8.memory[chip8.cpu.pc + 1];
            chip8.cpu.pc += 2;
            executeInstruction(opcode);
        }
    }

    if (!chip8.paused) {
        updateTimers();
    }

    updateDisplay(chip8.display);
    requestAnimationFrame(runCycle);
}

function executeInstruction(opcode) {
    const x = (opcode & 0x0F00) >> 8;
    const y = (opcode & 0x00F0) >> 4;
    const kk = opcode & 0x00FF;
    const nnn = opcode & 0x0FFF;
    const n = opcode & 0x000F;

    switch (opcode & 0xF000) {
        case 0x0000:
            switch (kk) {
                case 0xE0:
                    chip8.display.fill(0);
                    break;
                // Add 00EE RET if needed later
            }
            break;
        case 0x1000: // 1nnn - JP addr
            chip8.cpu.pc = nnn;
            break;
        case 0x6000: // 6xkk - LD Vx, byte
            chip8.cpu.register[x] = kk;
            break;
        case 0x7000: // 7xkk - ADD Vx, byte
            chip8.cpu.register[x] += kk;
            break;
        case 0xA000: // Annn - LD I, addr
            chip8.cpu.i = nnn;
            break;
        case 0xD000: // Dxyn - DRW Vx, Vy, nibble
            chip8.cpu.register[0xF] = 0;
            for (let row = 0; row < n; row++) {
                const spriteByte = chip8.memory[chip8.cpu.i + row];
                for (let col = 0; col < 8; col++) {
                    if ((spriteByte & (0x80 >> col)) !== 0) {
                        const screenX = (chip8.cpu.register[x] + col) % 64;
                        const screenY = (chip8.cpu.register[y] + row) % 32;
                        const pixelIndex = screenX + (screenY * 64);
                        if (chip8.display[pixelIndex] === 1) {
                            chip8.cpu.register[0xF] = 1;
                        }
                        chip8.display[pixelIndex] ^= 1;
                    }
                }
            }
            break;
    }
}
```



## Merging them all together

```ts
const chip8 = {
    memory: new Uint8Array(4096),
    cpu: {
        register: new Uint8Array(16),
        stack: new Uint16Array(16),
        sp: 0,
        pc: 0x200,
        i: 0
    },
    input: Array.from({ length: 16 }, () => false),
    clock: {
        delayTimer: 0,
        soundTimer: 0
    },
    display: Array.from({ length: 64 * 32 }, () => 0),
    paused: false,
    speed: 10
};

const canvas = document.createElement('canvas');
canvas.width = 64 * 10;
canvas.height = 32 * 10;
document.body.appendChild(canvas);
const ctx = canvas.getContext('2d');

function updateDisplay(display) {
    ctx.fillStyle = '#000';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    const scaleX = canvas.width / 64;
    const scaleY = canvas.height / 32;
    for (let i = 0; i < display.length; i++) {
        if (display[i]) {
            ctx.fillStyle = '#0F0';
            const x = (i % 64) * scaleX;
            const y = Math.floor(i / 64) * scaleY;
            ctx.fillRect(x, y, scaleX, scaleY);
        }
    }
}

function loadProgram(program) {
    for (let i = 0; i < program.length; i++) {
        chip8.memory[0x200 + i] = program[i];
    }
}

function runCycle() {
    for (let i = 0; i < chip8.speed; i++) {
        if (!chip8.paused) {
            const opcode = (chip8.memory[chip8.cpu.pc] << 8) | chip8.memory[chip8.cpu.pc + 1];
            chip8.cpu.pc += 2;
            executeInstruction(opcode);
        }
    }

    if (!chip8.paused) {
        updateTimers();
    }

    updateDisplay(chip8.display);
    requestAnimationFrame(runCycle);
}

function updateTimers() {
    if (chip8.clock.delayTimer > 0) {
        chip8.clock.delayTimer--;
    }
    if (chip8.clock.soundTimer > 0) {
        chip8.clock.soundTimer--;
    }
}

function executeInstruction(opcode) {
    const x = (opcode & 0x0F00) >> 8;
    const y = (opcode & 0x00F0) >> 4;
    const kk = opcode & 0x00FF;
    const nnn = opcode & 0x0FFF;
    const n = opcode & 0x000F;

    switch (opcode & 0xF000) {
        case 0x0000:
            switch (kk) {
                case 0xE0:
                    chip8.display.fill(0);
                    break;
                // Add 00EE RET if needed later
            }
            break;
        case 0x1000: // 1nnn - JP addr
            chip8.cpu.pc = nnn;
            break;
        case 0x6000: // 6xkk - LD Vx, byte
            chip8.cpu.register[x] = kk;
            break;
        case 0x7000: // 7xkk - ADD Vx, byte
            chip8.cpu.register[x] += kk;
            break;
        case 0xA000: // Annn - LD I, addr
            chip8.cpu.i = nnn;
            break;
        case 0xD000: // Dxyn - DRW Vx, Vy, nibble
            chip8.cpu.register[0xF] = 0;
            for (let row = 0; row < n; row++) {
                const spriteByte = chip8.memory[chip8.cpu.i + row];
                for (let col = 0; col < 8; col++) {
                    if ((spriteByte & (0x80 >> col)) !== 0) {
                        const screenX = (chip8.cpu.register[x] + col) % 64;
                        const screenY = (chip8.cpu.register[y] + row) % 32;
                        const pixelIndex = screenX + (screenY * 64);
                        if (chip8.display[pixelIndex] === 1) {
                            chip8.cpu.register[0xF] = 1;
                        }
                        chip8.display[pixelIndex] ^= 1;
                    }
                }
            }
            break;
    }
}

const helloProgram = [
    0x00, 0xE0, 0x60, 0x0A, 0x61, 0x0A, 0xA2, 0x24,
    0xD0, 0x15, 0x70, 0x06, 0xA2, 0x29, 0xD0, 0x15,
    0x70, 0x06, 0xA2, 0x2E, 0xD0, 0x15, 0x70, 0x06,
    0xA2, 0x2E, 0xD0, 0x15, 0x70, 0x06, 0xA2, 0x33,
    0xD0, 0x15, 0x12, 0x22, 0x88, 0x88, 0xF8, 0x88,
    0x88, 0xF8, 0x80, 0xF8, 0x80, 0xF8, 0x80, 0x80,
    0x80, 0x80, 0xF8, 0xF8, 0x88, 0x88, 0x88, 0xF8
];


loadProgram(helloProgram);
requestAnimationFrame(runCycle);
```


[code_explain]

Here I have merge all the previous code into one!

Alright everyone, let's look at the actual Chip-8 code that makes "HELLO" appear. Remember, Chip-8 instructions, or opcodes, are two bytes long.

1.  `0x00, 0xE0`
    *   This combines to the opcode `00E0`.
    *   Its job is **CLS** - Clear Screen. Just wipes the display clean so we start fresh.

2.  `0x60, 0x0A`
    *   Opcode `600A`.
    *   **LD V0, 0x0A**. This means "Load the value 10 (which is 0A in hexadecimal) into register V0".
    *   V0 will store the X-coordinate where we start drawing. So, we're starting 10 pixels from the left edge.

3.  `0x61, 0x0A`
    *   Opcode `610A`.
    *   **LD V1, 0x0A**. Similar to the last one, this loads the value 10 into register V1.
    *   V1 holds the Y-coordinate. So, we're also starting 10 pixels down from the top.

4.  `0xA2, 0x24`
    *   Opcode `A224`.
    *   **LD I, 0x224**. This loads the memory address `0x224` into the special register `I`.
    *   Register `I` is used to point to sprite data in memory. This specific address *should* point to the start of the pixel data for the letter 'H'. *(Self-correction based on previous thought: Explain the intent, not the potentially flawed address)* The emulator will look at this address to find the pixels for 'H'.

5.  `0xD0, 0x15`
    *   Opcode `D015`.
    *   **DRW V0, V1, 5**. This is the draw command! It means "Draw a sprite at coordinates (V0, V1) using 5 bytes of data starting from the address currently in register I".
    *   It reads 5 bytes starting from where `I` points (the 'H' data), interprets them as rows of pixels (8 pixels wide), and draws them onto the screen at the coordinates we set earlier (10, 10). It also checks for pixel collisions.

6.  `0x70, 0x06`
    *   Opcode `7006`.
    *   **ADD V0, 6**. This adds the value 6 to register V0 (our X coordinate).
    *   Since our characters are about 4 or 5 pixels wide, adding 6 moves our drawing position to the right, leaving a small gap before the next letter.

7.  `0xA2, 0x29`
    *   Opcode `A229`.
    *   **LD I, <address>**. Just like before, this loads a new address into `I`. This time, it's pointing to the sprite data for the letter 'E'.

8.  `0xD0, 0x15`
    *   Opcode `D015`.
    *   **DRW V0, V1, 5**. Draw again! Using the *new* X coordinate from V0 and the *same* Y coordinate in V1, it draws the 'E' sprite found at the address now in `I`.

9.  `0x70, 0x06`
    *   Opcode `7006`.
    *   **ADD V0, 6**. Move the X position right again, ready for the next letter.

10. `0xA2, 0x2E`
    *   Opcode `A22E`.
    *   **LD I, <address>**. Load the address for the 'L' sprite data into `I`.

11. `0xD0, 0x15`
    *   Opcode `D015`.
    *   **DRW V0, V1, 5**. Draw the 'L' sprite.

12. `0x70, 0x06`
    *   Opcode `7006`.
    *   **ADD V0, 6**. Move X right again.

13. `0xA2, 0x2E`
    *   Opcode `A22E`.
    *   **LD I, <address>**. Load the address for the 'L' sprite data into `I` *again*. We need two L's for "HELLO".

14. `0xD0, 0x15`
    *   Opcode `D015`.
    *   **DRW V0, V1, 5**. Draw the second 'L' sprite.

15. `0x70, 0x06`
    *   Opcode `7006`.
    *   **ADD V0, 6**. Move X right one last time.

16. `0xA2, 0x33`
    *   Opcode `A233`.
    *   **LD I, <address>**. Load the address for the 'O' sprite data into `I`.

17. `0xD0, 0x15`
    *   Opcode `D015`.
    *   **DRW V0, V1, 5**. Draw the 'O' sprite. Now we have "HELLO" on screen!

18. `0x12, 0x22`
    *   Opcode `1222`.
    *   **JP 0x222**. This means "Jump to address 0x222".
    *   This makes the program execution loop back to an earlier point (or just repeat this jump). It effectively halts the program here, keeping "HELLO" displayed indefinitely instead of running off the end of the code.

19. **The rest of the data (`0x88, 0x88, ... 0xF8`)**
    *   This block of bytes isn't instructions; it's the actual pixel data for the letters H, E, L, and O.
    *   Each sequence of 5 bytes represents one character. For example, `0x88, 0x88, 0xF8, 0x88, 0x88` is 'H'.
    *   Each byte represents a row of 8 pixels. For instance, `0x88` in binary is `10001000`, which looks like `*   *   ` on screen. `0xF8` is `11111000` or `*****   `. Stack 5 of these rows vertically, and you get the character shape!
    *   The `LD I` instructions we saw earlier were pointing to the start of each 5-byte chunk within this data block.

So, that's how this little program uses basic Chip-8 commands to clear the screen, set coordinates, point to pixel data, draw each letter, move the cursor, and finally loop to keep the result visible!

[end]