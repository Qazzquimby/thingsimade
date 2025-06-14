The goal is to make a moving masonry effect for a website.

Below are a work in progress implementation of the page, and a working python prototype of the animation generator.
Using the below, generate working svelte implementation.


# old WIP of page


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Neon Grid Masonry</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: #0a0a0a;
            font-family: 'Arial', sans-serif;
            padding: 20px;
            min-height: 100vh;
        }

        .grid-container {
            display: grid;
            grid-template-columns: repeat(8, 1fr);
            grid-auto-rows: 120px;
            gap: 12px;
            max-width: 1200px;
            margin: 0 auto;
        }

        .card {
            position: relative;
            cursor: pointer;
            transform-style: preserve-3d;
            transition: transform 0.6s ease;
            perspective: 1000px;
            border-radius: 12px;
        }

        .card-inner {
            position: relative;
            width: 100%;
            height: 100%;
            transform-style: preserve-3d;
            transition: transform 0.6s ease;
        }

        .card:hover .card-inner {
            transform: rotateY(180deg);
        }

        .card-front, .card-back {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            border-radius: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }

        .card-front {
            background: transparent;
            border: 3px solid;
            border-image: linear-gradient(45deg, #ff006e, #8338ec, #3a86ff, #06ffa5) 1;
            border-image-slice: 1;
            position: relative;
            overflow: hidden;
        }

        .card-front::before {
            content: '';
            position: absolute;
            top: -3px;
            left: -3px;
            right: -3px;
            bottom: -3px;
            background: linear-gradient(45deg, #ff006e, #8338ec, #3a86ff, #06ffa5);
            background-size: 400% 400%;
            border-radius: 12px;
            z-index: -1;
            animation: gradientShift 4s ease infinite;
        }

        .card-front::after {
            content: '';
            position: absolute;
            top: 3px;
            left: 3px;
            right: 3px;
            bottom: 3px;
            background: #0a0a0a;
            border-radius: 9px;
            z-index: -1;
        }

        .card-back {
            background: white;
            color: #333;
            transform: rotateY(180deg);
            text-align: center;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
        }

        .card-back h3 {
            font-size: 1.1em;
            margin-bottom: 8px;
            color: #2d2d2d;
        }

        .card-back p {
            font-size: 0.8em;
            line-height: 1.4;
            color: #666;
        }

        /* Grid span classes */
        .span-1x1 { grid-column: span 1; grid-row: span 1; }
        .span-2x1 { grid-column: span 2; grid-row: span 1; }
        .span-1x2 { grid-column: span 1; grid-row: span 2; }
        .span-2x2 { grid-column: span 2; grid-row: span 2; }

        @keyframes gradientShift {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }

        /* Neon glow effect */
        .card {
            filter: drop-shadow(0 0 10px rgba(255, 0, 110, 0.3)) 
                    drop-shadow(0 0 20px rgba(131, 56, 236, 0.2));
        }

        /* Animation transitions */
        .card.animating {
            transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
        }

        /* Responsive design */
        @media (max-width: 768px) {
            .grid-container {
                grid-template-columns: repeat(4, 1fr);
                grid-auto-rows: 100px;
            }
            
            .span-2x1, .span-2x2 {
                grid-column: span 2;
            }
        }

        @media (max-width: 480px) {
            .grid-container {
                grid-template-columns: repeat(2, 1fr);
                grid-auto-rows: 80px;
            }
            
            .span-2x1, .span-1x2, .span-2x2 {
                grid-column: span 1;
                grid-row: span 1;
            }
        }
    </style>
</head>
<body>
    <div class="grid-container" id="gridContainer">
        <!-- Cards will be generated by JavaScript -->
    </div>

    <script>
        const cardData = [
            { title: "Neon Dreams", text: "Vibrant colors dance in the digital night." },
            { title: "Tech Innovation", text: "Cutting-edge solutions for tomorrow." },
            { title: "Creative Vision", text: "Where imagination meets reality." },
            { title: "Digital Art", text: "Pixels transformed into masterpieces." },
            { title: "Future Design", text: "Tomorrow's aesthetics today." },
            { title: "Cyber Space", text: "Navigate endless possibilities." },
            { title: "Electric Pulse", text: "Feel the digital heartbeat." },
            { title: "Neon Nights", text: "Darkness becomes a canvas for light." },
            { title: "Code Poetry", text: "Beautiful algorithms in light." },
            { title: "Pixel Perfect", text: "Every detail crafted with care." },
            { title: "Glitch Art", text: "Beauty in digital imperfection." },
            { title: "Synthwave", text: "Retro-futuristic modern design." },
            { title: "Data Flow", text: "Information through colorful channels." },
            { title: "Holographic", text: "3D experiences in 2D space." },
            { title: "Binary Beauty", text: "1s and 0s, infinite possibilities." },
            { title: "Circuit Dreams", text: "Electronic pathways to new worlds." },
            { title: "Laser Focus", text: "Precision beams through darkness." },
            { title: "Quantum Leap", text: "Bridging realities with style." },
            { title: "Chrome Reflection", text: "Mirrors reflecting digital souls." },
            { title: "Neon Genesis", text: "Birth of new visual language." },
            { title: "Cyber Grid", text: "Connected nodes of possibility." },
            { title: "Digital Zen", text: "Finding peace in the matrix." },
            { title: "Neon Flow", text: "Energy streams through the void." },
            { title: "Tech Dreams", text: "Where silicon meets imagination." }
        ];

        const spanClasses = ['span-1x1', 'span-2x1', 'span-1x2', 'span-2x2'];
        const weights = [0.5, 0.25, 0.15, 0.1]; // Probability weights for different sizes
        const container = document.getElementById('gridContainer');

        function getRandomSpan() {
            const random = Math.random();
            let cumulativeWeight = 0;
            
            for (let i = 0; i < weights.length; i++) {
                cumulativeWeight += weights[i];
                if (random < cumulativeWeight) {
                    return spanClasses[i];
                }
            }
            return spanClasses[0]; // fallback
        }

        function generateCards() {
            cardData.forEach((data, index) => {
                const card = document.createElement('div');
                const spanClass = getRandomSpan();
                
                card.className = `card ${spanClass}`;
                card.dataset.cardId = index;
                card.innerHTML = `
                    <div class="card-inner">
                        <div class="card-front"></div>
                        <div class="card-back">
                            <h3>${data.title}</h3>
                            <p>${data.text}</p>
                        </div>
                    </div>
                `;
                
                container.appendChild(card);
            });
        }

        // Mutation system
        function getCardSize(card) {
            const classes = card.className.split(' ');
            for (let cls of classes) {
                if (cls.startsWith('span-')) {
                    const [w, h] = cls.split('-')[1].split('x').map(Number);
                    return { width: w, height: h, class: cls };
                }
            }
            return { width: 1, height: 1, class: 'span-1x1' };
        }

        function setCardSize(card, newClass) {
            card.className = card.className.replace(/span-\d+x\d+/, newClass);
        }

        function findMutationOpportunity() {
            const cards = Array.from(container.children);
            const mutations = [];

            // Find transformation opportunities
            for (let i = 0; i < cards.length - 1; i++) {
                const card1 = cards[i];
                const card2 = cards[i + 1];
                const size1 = getCardSize(card1);
                const size2 = getCardSize(card2);

                // Example mutation: 2x2 + 1x1 -> 1x2 + 2x1
                if (size1.width === 2 && size1.height === 2 && size2.width === 1 && size2.height === 1) {
                    mutations.push({
                        type: 'shrink_grow',
                        cards: [card1, card2],
                        transforms: [
                            { card: card1, from: size1.class, to: 'span-1x2' },
                            { card: card2, from: size2.class, to: 'span-2x1' }
                        ]
                    });
                }

                // 2x1 + 1x1 -> 1x1 + 2x1
                if (size1.width === 2 && size1.height === 1 && size2.width === 1 && size2.height === 1) {
                    mutations.push({
                        type: 'redistribute',
                        cards: [card1, card2],
                        transforms: [
                            { card: card1, from: size1.class, to: 'span-1x1' },
                            { card: card2, from: size2.class, to: 'span-2x1' }
                        ]
                    });
                }

                // 1x2 + 1x1 -> 1x1 + 1x2
                if (size1.width === 1 && size1.height === 2 && size2.width === 1 && size2.height === 1) {
                    mutations.push({
                        type: 'height_swap',
                        cards: [card1, card2],
                        transforms: [
                            { card: card1, from: size1.class, to: 'span-1x1' },
                            { card: card2, from: size2.class, to: 'span-1x2' }
                        ]
                    });
                }

                // Reverse transformations
                if (size1.width === 1 && size1.height === 2 && size2.width === 2 && size2.height === 1) {
                    mutations.push({
                        type: 'expand_combine',
                        cards: [card1, card2],
                        transforms: [
                            { card: card1, from: size1.class, to: 'span-2x2' },
                            { card: card2, from: size2.class, to: 'span-1x1' }
                        ]
                    });
                }
            }

            return mutations;
        }

        async function animateTransformation(transforms) {
            // Add animation class to all affected cards
            transforms.forEach(t => {
                t.card.classList.add('animating');
            });

            // Apply transformations in sequence with delays
            for (let i = 0; i < transforms.length; i++) {
                setTimeout(() => {
                    setCardSize(transforms[i].card, transforms[i].to);
                }, i * 200);
            }

            // Remove animation class after completion
            setTimeout(() => {
                transforms.forEach(t => {
                    t.card.classList.remove('animating');
                });
            }, transforms.length * 200 + 800);
        }

        function performMutation() {
            const opportunities = findMutationOpportunity();
            if (opportunities.length > 0) {
                const chosen = opportunities[Math.floor(Math.random() * opportunities.length)];
                animateTransformation(chosen.transforms);
                console.log(`Performing mutation: ${chosen.type}`);
            }
        }

        // Initialize grid and start mutation cycle
        generateCards();
        
        // Start the mutation cycle
        setInterval(performMutation, 5000);
    </script>
</body>
</html>


# Working python prototype of animation generator

import random
import string
import copy
from dataclasses import dataclass, field
from typing import List, Tuple, Any, Union


@dataclass
class ShrinkDetail:
    new_x: int
    new_y: int
    new_w: int
    new_h: int
    gap_coords: List[Tuple[int, int]]


@dataclass
class FillAction:
    action_type: str  # 'translate' or 'resize'
    target_x: int
    target_y: int
    target_w: int
    target_h: int


@dataclass
class PossibleFill:
    piece_id: Any
    fill_action: FillAction


@dataclass
class ShrinkActionLog:
    action: str = field(default="shrink", init=False)
    piece_id: Any
    original_metrics: Tuple[int, int, int, int]  # x, y, w, h
    applied_shrink_detail: ShrinkDetail

    @property
    def new_pos(self) -> Tuple[int, int]:
        return self.applied_shrink_detail.new_x, self.applied_shrink_detail.new_y

    @property
    def new_size(self) -> Tuple[int, int]:
        return self.applied_shrink_detail.new_w, self.applied_shrink_detail.new_h

    @property
    def gap_created(self) -> List[Tuple[int, int]]:
        return self.applied_shrink_detail.gap_coords


@dataclass
class FillActionLog:
    action: str  # 'translate' or 'resize'
    piece_id: Any
    new_pos: Tuple[int, int]
    new_size: Tuple[int, int]


ActionLog = Union[ShrinkActionLog, FillActionLog]


class Piece:
    def __init__(self, id, x, y, width, height):
        self.id = id
        self.x = x  # top-left x
        self.y = y  # top-left y
        self.width = width
        self.height = height

    def __repr__(self):
        return (
            f"Piece({self.id}, x={self.x}, y={self.y}, w={self.width}, h={self.height})"
        )


class Grid:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.pieces = {}  # id -> Piece object
        # board[row][col] stores piece_id or None
        self.board = [[None for _ in range(width)] for _ in range(height)]
        self._next_piece_id_counter = 1
        self._piece_chars = {}  # id -> char for drawing

    def clone(self):
        new_grid = Grid(self.width, self.height)
        new_grid.pieces = copy.deepcopy(self.pieces)
        new_grid.board = copy.deepcopy(self.board)
        new_grid._next_piece_id_counter = self._next_piece_id_counter
        new_grid._piece_chars = copy.deepcopy(self._piece_chars)
        return new_grid

    def _get_next_piece_id(self) -> int:
        pid = self._next_piece_id_counter
        self._next_piece_id_counter += 1
        return pid

    def _assign_char(self, piece_id):
        if piece_id not in self._piece_chars:
            available_chars = (
                string.ascii_uppercase + string.ascii_lowercase + string.digits
            )

            # Find next available char
            current_assigned_chars = set(self._piece_chars.values())
            chosen_char = None
            for char_candidate in available_chars:
                if char_candidate not in current_assigned_chars:
                    chosen_char = char_candidate
                    break
            if chosen_char is None:  # Fallback if all standard chars are used
                # Start numbering fallback chars from a point that doesn't clash with single digits
                fallback_idx = 0
                while True:
                    chosen_char = (
                        f"@{fallback_idx}"  # Using @ to avoid confusion with piece IDs
                    )
                    if chosen_char not in current_assigned_chars:
                        break
                    fallback_idx += 1

            self._piece_chars[piece_id] = chosen_char
        return self._piece_chars[piece_id]

    def add_piece(self, x, y, width, height, piece_id=None):
        if piece_id is None:
            piece_id = self._get_next_piece_id()

        if not (
            0 <= x < self.width
            and 0 <= y < self.height
            and x + width <= self.width
            and y + height <= self.height
        ):
            raise ValueError(
                f"Piece {piece_id} ({x},{y} {width}x{height}) out of bounds for grid {self.width}x{self.height}"
            )

        for r in range(y, y + height):
            for c in range(x, x + width):
                if self.board[r][c] is not None:
                    raise ValueError(
                        f"Cell ({c}, {r}) is already occupied by piece {self.board[r][c]} when trying to place {piece_id}"
                    )

        piece = Piece(piece_id, x, y, width, height)
        self.pieces[piece_id] = piece
        self._assign_char(piece_id)  # Ensure char is assigned for drawing

        for r in range(y, y + height):
            for c in range(x, x + width):
                self.board[r][c] = piece_id
        return piece

    def _clear_piece_from_board(self, piece):
        for r in range(piece.y, piece.y + piece.height):
            for c in range(piece.x, piece.x + piece.width):
                if self.board[r][c] == piece.id:
                    self.board[r][c] = None

    def _place_piece_on_board(self, piece):
        for r in range(piece.y, piece.y + piece.height):
            for c in range(piece.x, piece.x + piece.width):
                # This check helps catch issues if a cell is unexpectedly occupied
                # if self.board[r][c] is not None:
                #     print(f"Warning: Cell ({c},{r}) occupied by {self.board[r][c]} when placing {piece.id}")
                self.board[r][c] = piece.id

    def draw(self):
        # Create a border string like "+-+-+-+"
        border = "+" + "+".join(["-"] * self.width) + "+"
        print(border)
        for r in range(self.height):
            row_str = "|"
            for c in range(self.width):
                piece_id = self.board[r][c]
                if piece_id is not None:
                    row_str += self._piece_chars.get(piece_id, "?") + "|"
                else:
                    row_str += " |"  # Empty space
            print(row_str)
            print(border)

    def find_empty_spaces(self):
        empty_spaces = []
        for r in range(self.height):
            for c in range(self.width):
                if self.board[r][c] is None:
                    empty_spaces.append((c, r))  # x, y
        return empty_spaces

    def is_fully_tiled(self):
        return not self.find_empty_spaces()

    def get_shrinkable_pieces(self):
        shrinkable = []
        for piece in self.pieces.values():
            if piece.width > 1 or piece.height > 1:  # Must be larger than 1x1
                # Check if it's one of the standard piece types that can be shrunk
                if (
                    (piece.width == 1 and piece.height == 2)
                    or (piece.width == 2 and piece.height == 1)
                    or (piece.width == 2 and piece.height == 2)
                ):
                    shrinkable.append(piece)
        return shrinkable

    def get_possible_shrinks(self, piece_id):
        if piece_id not in self.pieces:
            raise ValueError(f"Piece {piece_id} not found.")

        piece = self.pieces[piece_id]
        original_x, original_y = piece.x, piece.y
        original_width, original_height = piece.width, piece.height

        if original_width == 1 and original_height == 1:
            return []

        possible_shrinks_details = []
        # 1x2 becomes a top or bottom 1x1
        if original_width == 1 and original_height == 2:
            possible_shrinks_details.append(
                ShrinkDetail(
                    new_x=original_x,
                    new_y=original_y,
                    new_w=1,
                    new_h=1,
                    gap_coords=[(original_x, original_y + 1)],
                )
            )
            possible_shrinks_details.append(
                ShrinkDetail(
                    new_x=original_x,
                    new_y=original_y + 1,
                    new_w=1,
                    new_h=1,
                    gap_coords=[(original_x, original_y)],
                )
            )
        # 2x1 becomes a left or right 1x1
        elif original_width == 2 and original_height == 1:
            possible_shrinks_details.append(
                ShrinkDetail(
                    new_x=original_x,
                    new_y=original_y,
                    new_w=1,
                    new_h=1,
                    gap_coords=[(original_x + 1, original_y)],
                )
            )
            possible_shrinks_details.append(
                ShrinkDetail(
                    new_x=original_x + 1,
                    new_y=original_y,
                    new_w=1,
                    new_h=1,
                    gap_coords=[(original_x, original_y)],
                )
            )
        # 2x2 becomes a left/right 1x2 or a top/bottom 2x1
        elif original_width == 2 and original_height == 2:
            possible_shrinks_details.append(
                ShrinkDetail(
                    new_x=original_x,
                    new_y=original_y,
                    new_w=2,
                    new_h=1,
                    gap_coords=[
                        (original_x, original_y + 1),
                        (original_x + 1, original_y + 1),
                    ],
                )
            )
            possible_shrinks_details.append(
                ShrinkDetail(
                    new_x=original_x,
                    new_y=original_y + 1,
                    new_w=2,
                    new_h=1,
                    gap_coords=[
                        (original_x, original_y),
                        (original_x + 1, original_y),
                    ],
                )
            )
            possible_shrinks_details.append(
                ShrinkDetail(
                    new_x=original_x,
                    new_y=original_y,
                    new_w=1,
                    new_h=2,
                    gap_coords=[
                        (original_x + 1, original_y),
                        (original_x + 1, original_y + 1),
                    ],
                )
            )
            possible_shrinks_details.append(
                ShrinkDetail(
                    new_x=original_x + 1,
                    new_y=original_y,
                    new_w=1,
                    new_h=2,
                    gap_coords=[
                        (original_x, original_y),
                        (original_x, original_y + 1),
                    ],
                )
            )
        return possible_shrinks_details

    def apply_shrink(
        self, piece_id: Any, shrink_details: ShrinkDetail
    ) -> List[Tuple[int, int]]:
        if piece_id not in self.pieces:
            raise ValueError(f"Piece {piece_id} not found for shrink.")
        piece = self.pieces[piece_id]

        self._clear_piece_from_board(piece)

        piece.x = shrink_details.new_x
        piece.y = shrink_details.new_y
        piece.width = shrink_details.new_w
        piece.height = shrink_details.new_h

        self._place_piece_on_board(piece)
        return shrink_details.gap_coords

    def get_adjacent_pieces_to_gap(
        self, gap_coords: List[Tuple[int, int]]
    ) -> List[Piece]:
        adjacent_piece_ids = set()
        for gx, gy in gap_coords:
            for dx, dy in [(0, -1), (0, 1), (-1, 0), (1, 0)]:  # N, S, W, E
                nx, ny = gx + dx, gy + dy
                if 0 <= nx < self.width and 0 <= ny < self.height:
                    piece_id_at_neighbor = self.board[ny][nx]
                    if piece_id_at_neighbor is not None:
                        adjacent_piece_ids.add(piece_id_at_neighbor)
        return [self.pieces[pid] for pid in adjacent_piece_ids]

    def get_fill_actions_for_piece(
        self, piece: Piece, gap_coords: List[Tuple[int, int]]
    ) -> List[FillAction]:
        possible_actions: List[FillAction] = []

        min_gx = min(c[0] for c in gap_coords)
        max_gx = max(c[0] for c in gap_coords)
        min_gy = min(c[1] for c in gap_coords)
        max_gy = max(c[1] for c in gap_coords)

        gap_width = max_gx - min_gx + 1
        gap_height = max_gy - min_gy + 1

        if len(gap_coords) != (gap_width * gap_height):  # Gap not rectangular
            assert False

        # Try Translate
        if (
            piece.width == gap_width and piece.height == gap_height
        ):  # Exact match for translate
            # Piece is left of gap, moves right
            if piece.x + piece.width == min_gx and piece.y == min_gy:
                possible_actions.append(
                    FillAction("translate", min_gx, min_gy, piece.width, piece.height)
                )
            # Piece is right of gap, moves left
            if piece.x == max_gx + 1 and piece.y == min_gy:
                possible_actions.append(
                    FillAction("translate", min_gx, min_gy, piece.width, piece.height)
                )
            # Piece is above gap, moves down
            if piece.y + piece.height == min_gy and piece.x == min_gx:
                possible_actions.append(
                    FillAction("translate", min_gx, min_gy, piece.width, piece.height)
                )
            # Piece is below gap, moves up
            if piece.y == max_gy + 1 and piece.x == min_gx:
                possible_actions.append(
                    FillAction("translate", min_gx, min_gy, piece.width, piece.height)
                )

        # Try Resize (Expand)
        # Piece expands in width
        if (
            piece.y == min_gy and piece.height == gap_height
        ):  # Vertically aligned with gap
            if piece.x + piece.width == min_gx:  # Piece to left of gap
                new_w = piece.width + gap_width
                if new_w <= 2 and piece.height <= 2:
                    possible_actions.append(
                        FillAction("resize", piece.x, piece.y, new_w, piece.height)
                    )
            elif piece.x == max_gx + 1:  # Piece to right of gap
                new_w = piece.width + gap_width
                if new_w <= 2 and piece.height <= 2:
                    possible_actions.append(
                        FillAction("resize", min_gx, piece.y, new_w, piece.height)
                    )
        # Piece expands in height
        if (
            piece.x == min_gx and piece.width == gap_width
        ):  # Horizontally aligned with gap
            if piece.y + piece.height == min_gy:  # Piece above gap
                new_h = piece.height + gap_height
                if piece.width <= 2 and new_h <= 2:
                    possible_actions.append(
                        FillAction("resize", piece.x, piece.y, piece.width, new_h)
                    )
            elif piece.y == max_gy + 1:  # Piece below gap
                new_h = piece.height + gap_height
                if piece.width <= 2 and new_h <= 2:
                    possible_actions.append(
                        FillAction("resize", piece.x, min_gy, piece.width, new_h)
                    )

        return possible_actions

    def apply_fill_action(
        self,
        piece_id: Any,
        new_x: int,
        new_y: int,
        new_width: int,
        new_height: int,
    ):
        piece = self.pieces[piece_id]
        self._clear_piece_from_board(piece)

        piece.x = new_x
        piece.y = new_y
        piece.width = new_width
        piece.height = new_height

        self._place_piece_on_board(piece)


def generate_random_tiling(grid_width, grid_height, max_attempts=100):
    for _attempt in range(max_attempts):
        grid = Grid(grid_width, grid_height)
        # Piece types: (width, height), try to place larger pieces first generally
        piece_options_ordered = [(2, 2), (2, 1), (1, 2), (1, 1)]

        # Iterate through each cell and try to place a piece if it's empty
        for r_start in range(grid_height):
            for c_start in range(grid_width):
                if grid.board[r_start][c_start] is None:  # If cell is empty

                    current_piece_options = list(piece_options_ordered)
                    random.shuffle(current_piece_options)

                    placed_this_cell = False
                    for p_w, p_h in current_piece_options:
                        can_place = True
                        if not (
                            0 <= c_start < grid_width
                            and 0 <= r_start < grid_height
                            and c_start + p_w <= grid_width
                            and r_start + p_h <= grid_height
                        ):
                            can_place = False
                        else:
                            for r_offset in range(p_h):
                                for c_offset in range(p_w):
                                    if (
                                        grid.board[r_start + r_offset][
                                            c_start + c_offset
                                        ]
                                        is not None
                                    ):
                                        can_place = False
                                        break
                                if not can_place:
                                    break

                        if can_place:
                            grid.add_piece(c_start, r_start, p_w, p_h)
                            placed_this_cell = True
                            break  # Placed a piece for this cell's top-left

                    if not placed_this_cell and grid.board[r_start][c_start] is None:
                        assert False
                        # This means an empty cell could not be filled even by a 1x1,
                        # which indicates a logic error or impossible tiling state from this path.
                        # Break from this attempt and try a new grid generation.
            else:  # Inner loop (c_start) completed without break
                continue

        if grid.is_fully_tiled():
            return grid

    raise Exception(
        f"Failed to generate a random tiling for {grid_width}x{grid_height} after {max_attempts} attempts."
    )


def get_mutation_path(grid) -> List[ActionLog]:
    if not isinstance(grid, Grid):
        raise TypeError("Input must be a Grid object.")

    for _try_attempt in range(100):  # Max 100 tries to find a valid mutation path
        grid_copy = grid.clone()
        mutation_path: List[ActionLog] = []

        shrinkable_pieces_on_copy = grid_copy.get_shrinkable_pieces()
        if not shrinkable_pieces_on_copy:
            continue  # No shrinkable pieces on copy, try new random state (should not happen with valid initial grid)

        piece_to_shrink_on_copy = random.choice(shrinkable_pieces_on_copy)

        # Store original metrics of the piece *before* it's shrunk on the copy
        # These are relative to the copy's state, which is same as original_grid's state at this point
        pts_orig_x = piece_to_shrink_on_copy.x
        pts_orig_y = piece_to_shrink_on_copy.y
        pts_orig_w = piece_to_shrink_on_copy.width
        pts_orig_h = piece_to_shrink_on_copy.height

        possible_shrinks = grid_copy.get_possible_shrinks(piece_to_shrink_on_copy.id)
        if not possible_shrinks:
            continue  # Should not happen if get_shrinkable_pieces is correct

        chosen_shrink_details = random.choice(possible_shrinks)

        gap_coords = grid_copy.apply_shrink(
            piece_to_shrink_on_copy.id, chosen_shrink_details
        )

        shrunk_piece_on_copy = grid_copy.pieces[
            piece_to_shrink_on_copy.id
        ]  # piece_to_shrink_on_copy is updated by apply_shrink

        mutation_path.append(
            ShrinkActionLog(
                piece_id=piece_to_shrink_on_copy.id,
                original_metrics=(pts_orig_x, pts_orig_y, pts_orig_w, pts_orig_h),
                applied_shrink_detail=chosen_shrink_details,
            )
        )

        planning_succeeded = True
        for step_count in range(5):  # Max 5 fill steps
            if grid_copy.is_fully_tiled():
                break

            current_gap = grid_copy.find_empty_spaces()
            if not current_gap:  # Should be caught by is_fully_tiled
                planning_succeeded = False
                break

            adjacent_pieces_to_gap = grid_copy.get_adjacent_pieces_to_gap(current_gap)
            possible_fills: List[PossibleFill] = []

            for adj_piece_on_copy in adjacent_pieces_to_gap:
                fill_actions = grid_copy.get_fill_actions_for_piece(
                    adj_piece_on_copy, current_gap
                )
                for fill_action in fill_actions:
                    is_immediate_reversal = False
                    if (
                        step_count == 0
                        and adj_piece_on_copy.id == piece_to_shrink_on_copy.id
                        and fill_action.action_type == "resize"
                    ):
                        if (
                            fill_action.target_x == pts_orig_x
                            and fill_action.target_y == pts_orig_y
                            and fill_action.target_w == pts_orig_w
                            and fill_action.target_h == pts_orig_h
                        ):
                            is_immediate_reversal = True

                    if not is_immediate_reversal:
                        possible_fills.append(
                            PossibleFill(
                                piece_id=adj_piece_on_copy.id, fill_action=fill_action
                            )
                        )

            if not possible_fills:
                planning_succeeded = False
                break

            chosen_fill_plan = random.choice(possible_fills)

            cf_piece_id = chosen_fill_plan.piece_id
            cf_action = chosen_fill_plan.fill_action

            grid_copy.apply_fill_action(
                piece_id=cf_piece_id,
                new_x=cf_action.target_x,
                new_y=cf_action.target_y,
                new_width=cf_action.target_w,
                new_height=cf_action.target_h,
            )

            log_new_pos = (cf_action.target_x, cf_action.target_y)
            log_new_size = (cf_action.target_w, cf_action.target_h)

            if cf_action.action_type == "translate":
                # For translate, the piece's dimensions don't change, target_w/h reflect this.
                # The piece object on grid_copy is already updated.
                translated_piece = grid_copy.pieces[cf_piece_id]
                log_new_size = (translated_piece.width, translated_piece.height)

            mutation_path.append(
                FillActionLog(
                    action=cf_action.action_type,
                    piece_id=cf_piece_id,
                    new_pos=log_new_pos,
                    new_size=log_new_size,
                )
            )

        if planning_succeeded and grid_copy.is_fully_tiled():
            return mutation_path

    return []


def apply_mutation_path(grid, mutation_path):
    for action_log_item in mutation_path:
        if isinstance(action_log_item, ShrinkActionLog):
            grid.apply_shrink(
                action_log_item.piece_id, action_log_item.applied_shrink_detail
            )
        elif isinstance(action_log_item, FillActionLog):
            grid.apply_fill_action(
                piece_id=action_log_item.piece_id,
                new_x=action_log_item.new_pos[0],
                new_y=action_log_item.new_pos[1],
                new_width=action_log_item.new_size[0],
                new_height=action_log_item.new_size[1],
            )