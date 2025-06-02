<script>
  import { onMount } from 'svelte';

  // --- Constants and Configuration ---
  const GRID_WIDTH = 8; 
  const GRID_HEIGHT = 6; 
  const MUTATION_INTERVAL = 5000; // ms

  // --- Card Data (from HTML WIP, corrected typo) ---
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
  let cardDataIndex = 0;

  // --- Reactive State ---
  let pieces = []; 
  let board = []; // 2D array [row][col] -> piece_id or null
  
  // --- ID Generation ---
  let _next_piece_id_counter = 1;
  function _get_next_piece_id() {
    return _next_piece_id_counter++;
  }

  // --- Board and Piece Utilities ---
  function initializeBoard() {
    board = Array(GRID_HEIGHT).fill(null).map(() => Array(GRID_WIDTH).fill(null));
  }

  function _clear_piece_from_board_js(piece, currentBoard) {
    for (let r = piece.y; r < piece.y + piece.height; r++) {
      for (let c = piece.x; c < piece.x + piece.width; c++) {
        if (currentBoard[r] && currentBoard[r][c] === piece.id) {
          currentBoard[r][c] = null;
        }
      }
    }
  }

  function _place_piece_on_board_js(piece, currentBoard) {
    for (let r = piece.y; r < piece.y + piece.height; r++) {
      for (let c = piece.x; c < piece.x + piece.width; c++) {
        if (currentBoard[r]) currentBoard[r][c] = piece.id;
      }
    }
  }
  
  function getPieceById(id, currentPieces) {
    return currentPieces.find(p => p.id === id);
  }

  function updatePieceInPiecesArray(updatedPiece) { // Operates on global `pieces`
    const index = pieces.findIndex(p => p.id === updatedPiece.id);
    if (index !== -1) {
      pieces[index] = { ...pieces[index], ...updatedPiece };
      pieces = [...pieces]; // Ensure reactivity
    }
  }
  
  function generateInitialGrid() {
    initializeBoard(); // Operates on global `board`
    _next_piece_id_counter = 1;
    cardDataIndex = 0;
    let localPieces = [];

    const piece_options_ordered = [{w:2,h:2}, {w:2,h:1}, {w:1,h:2}, {w:1,h:1}];

    for (let r_start = 0; r_start < GRID_HEIGHT; r_start++) {
      for (let c_start = 0; c_start < GRID_WIDTH; c_start++) {
        if (board[r_start][c_start] === null) {
          let current_piece_options = [...piece_options_ordered];
          for (let i = current_piece_options.length - 1; i > 0; i--) { // Shuffle
            const j = Math.floor(Math.random() * (i + 1));
            [current_piece_options[i], current_piece_options[j]] = [current_piece_options[j], current_piece_options[i]];
          }

          let placed_this_cell = false;
          for (const p_opt of current_piece_options) {
            const p_w = p_opt.w;
            const p_h = p_opt.h;
            let can_place = true;

            if (!(c_start + p_w <= GRID_WIDTH && r_start + p_h <= GRID_HEIGHT)) {
              can_place = false;
            } else {
              for (let r_offset = 0; r_offset < p_h; r_offset++) {
                for (let c_offset = 0; c_offset < p_w; c_offset++) {
                  if (board[r_start + r_offset][c_start + c_offset] !== null) {
                    can_place = false; break;
                  }
                }
                if (!can_place) break;
              }
            }

            if (can_place) {
              const piece_id = _get_next_piece_id();
              const cardContent = cardData[cardDataIndex % cardData.length];
              cardDataIndex++;
              const newPiece = {
                id: piece_id, x: c_start, y: r_start, width: p_w, height: p_h,
                title: cardContent.title, text: cardContent.text, key: piece_id,
              };
              localPieces.push(newPiece);
              _place_piece_on_board_js(newPiece, board); // Use global board
              placed_this_cell = true;
              break;
            }
          }
          if (!placed_this_cell && board[r_start][c_start] === null) {
             console.error("Cell not filled, attempting 1x1 fallback:", c_start, r_start);
             const piece_id = _get_next_piece_id();
             const cardContent = cardData[cardDataIndex % cardData.length]; cardDataIndex++;
             const newPiece = { id: piece_id, x: c_start, y: r_start, width: 1, height: 1, title: cardContent.title, text: cardContent.text, key: piece_id };
             localPieces.push(newPiece);
             _place_piece_on_board_js(newPiece, board);
          }
        }
      }
    }
    pieces = localPieces;
    if (!is_fully_tiled_js(board)) {
        console.warn("Grid not fully tiled after generation. Retrying...");
        setTimeout(generateInitialGrid, 100); 
    }
  }

  function find_empty_spaces_js(currentBoard) {
    const empty_spaces = [];
    for (let r = 0; r < GRID_HEIGHT; r++) {
      for (let c = 0; c < GRID_WIDTH; c++) {
        if (currentBoard[r][c] === null) {
          empty_spaces.push({x: c, y: r});
        }
      }
    }
    return empty_spaces;
  }

  function is_fully_tiled_js(currentBoard) {
    return find_empty_spaces_js(currentBoard).length === 0;
  }

  function get_shrinkable_pieces_js(currentPieces) {
    return currentPieces.filter(p => 
      (p.width > 1 || p.height > 1) &&
      ((p.width === 1 && p.height === 2) || (p.width === 2 && p.height === 1) || (p.width === 2 && p.height === 2))
    );
  }

  function get_possible_shrinks_js(piece) {
    if (!piece) return [];
    const { x: ox, y: oy, width: ow, height: oh } = piece;
    if (ow === 1 && oh === 1) return [];
    
    const possible_shrinks_details = [];
    if (ow === 1 && oh === 2) { // 1x2 to 1x1
        possible_shrinks_details.push({ new_x: ox, new_y: oy, new_w: 1, new_h: 1, gap_coords: [{x: ox, y: oy + 1}] });
        possible_shrinks_details.push({ new_x: ox, new_y: oy + 1, new_w: 1, new_h: 1, gap_coords: [{x: ox, y: oy}] });
    } else if (ow === 2 && oh === 1) { // 2x1 to 1x1
        possible_shrinks_details.push({ new_x: ox, new_y: oy, new_w: 1, new_h: 1, gap_coords: [{x: ox + 1, y: oy}] });
        possible_shrinks_details.push({ new_x: ox + 1, new_y: oy, new_w: 1, new_h: 1, gap_coords: [{x: ox, y: oy}] });
    } else if (ow === 2 && oh === 2) { // 2x2
        possible_shrinks_details.push({ new_x: ox, new_y: oy, new_w: 2, new_h: 1, gap_coords: [{x: ox, y: oy + 1}, {x: ox + 1, y: oy + 1}] }); // to 2x1 top
        possible_shrinks_details.push({ new_x: ox, new_y: oy + 1, new_w: 2, new_h: 1, gap_coords: [{x: ox, y: oy}, {x: ox + 1, y: oy}] }); // to 2x1 bottom
        possible_shrinks_details.push({ new_x: ox, new_y: oy, new_w: 1, new_h: 2, gap_coords: [{x: ox + 1, y: oy}, {x: ox + 1, y: oy + 1}] }); // to 1x2 left
        possible_shrinks_details.push({ new_x: ox + 1, new_y: oy, new_w: 1, new_h: 2, gap_coords: [{x: ox, y: oy}, {x: ox, y: oy + 1}] }); // to 1x2 right
    }
    return possible_shrinks_details;
  }

  function apply_shrink_js_to_state(piece_id, shrink_details) { // Operates on global `pieces` and `board`
    let piece = getPieceById(piece_id, pieces);
    if (!piece) throw new Error(`Piece ${piece_id} not found for shrink.`);
    
    _clear_piece_from_board_js(piece, board);
    
    const updatedPiece = { ...piece, 
        x: shrink_details.new_x, y: shrink_details.new_y, 
        width: shrink_details.new_w, height: shrink_details.new_h 
    };
    
    _place_piece_on_board_js(updatedPiece, board);
    updatePieceInPiecesArray(updatedPiece);

    return shrink_details.gap_coords;
  }

  function get_adjacent_pieces_to_gap_js(gap_coords, currentBoard, currentPieces) {
    const adjacent_piece_ids = new Set();
    for (const gc of gap_coords) {
      const {x: gx, y: gy} = gc;
      for (const [dx, dy] of [[0, -1], [0, 1], [-1, 0], [1, 0]]) {
        const nx = gx + dx;
        const ny = gy + dy;
        if (nx >= 0 && nx < GRID_WIDTH && ny >= 0 && ny < GRID_HEIGHT) {
          const piece_id_at_neighbor = currentBoard[ny][nx];
          if (piece_id_at_neighbor !== null) {
            adjacent_piece_ids.add(piece_id_at_neighbor);
          }
        }
      }
    }
    return Array.from(adjacent_piece_ids).map(id => getPieceById(id, currentPieces)).filter(p => p);
  }

  function get_fill_actions_for_piece_js(piece, gap_coords) {
    const possible_actions = [];
    if (!piece || !gap_coords || gap_coords.length === 0) return possible_actions;

    const min_gx = Math.min(...gap_coords.map(c => c.x));
    const max_gx = Math.max(...gap_coords.map(c => c.x));
    const min_gy = Math.min(...gap_coords.map(c => c.y));
    const max_gy = Math.max(...gap_coords.map(c => c.y));

    const gap_width = max_gx - min_gx + 1;
    const gap_height = max_gy - min_gy + 1;

    if (gap_coords.length !== (gap_width * gap_height)) {
        console.error("Gap is not rectangular", gap_coords); return [];
    }

    if (piece.width === gap_width && piece.height === gap_height) {
        if (piece.x + piece.width === min_gx && piece.y === min_gy && piece.height === gap_height)
            possible_actions.push({ action_type: 'translate', target_x: min_gx, target_y: min_gy, target_w: piece.width, target_h: piece.height });
        if (piece.x === max_gx + 1 && piece.y === min_gy && piece.height === gap_height)
             possible_actions.push({ action_type: 'translate', target_x: min_gx, target_y: min_gy, target_w: piece.width, target_h: piece.height });
        if (piece.y + piece.height === min_gy && piece.x === min_gx && piece.width === gap_width)
            possible_actions.push({ action_type: 'translate', target_x: min_gx, target_y: min_gy, target_w: piece.width, target_h: piece.height });
        if (piece.y === max_gy + 1 && piece.x === min_gx && piece.width === gap_width)
            possible_actions.push({ action_type: 'translate', target_x: min_gx, target_y: min_gy, target_w: piece.width, target_h: piece.height });
    }
    
    if (piece.y === min_gy && piece.height === gap_height) {
        if (piece.x + piece.width === min_gx) { 
            const new_w = piece.width + gap_width;
            if (new_w <= 2 && piece.height <= 2) possible_actions.push({ action_type: 'resize', target_x: piece.x, target_y: piece.y, target_w: new_w, target_h: piece.height });
        } else if (piece.x === max_gx + 1) {
            const new_w = piece.width + gap_width;
            if (new_w <= 2 && piece.height <= 2) possible_actions.push({ action_type: 'resize', target_x: min_gx, target_y: piece.y, target_w: new_w, target_h: piece.height });
        }
    }
    if (piece.x === min_gx && piece.width === gap_width) {
        if (piece.y + piece.height === min_gy) {
            const new_h = piece.height + gap_height;
            if (piece.width <= 2 && new_h <= 2) possible_actions.push({ action_type: 'resize', target_x: piece.x, target_y: piece.y, target_w: piece.width, target_h: new_h });
        } else if (piece.y === max_gy + 1) {
            const new_h = piece.height + gap_height;
            if (piece.width <= 2 && new_h <= 2) possible_actions.push({ action_type: 'resize', target_x: piece.x, target_y: min_gy, target_w: piece.width, target_h: new_h });
        }
    }
    return possible_actions;
  }

  function apply_fill_action_js_to_state(piece_id, fill_action_details) { // Operates on global `pieces` and `board`
    let piece = getPieceById(piece_id, pieces);
    if (!piece) throw new Error(`Piece ${piece_id} not found for fill.`);

    _clear_piece_from_board_js(piece, board);
    const updatedPiece = { ...piece,
        x: fill_action_details.target_x, y: fill_action_details.target_y,
        width: fill_action_details.target_w, height: fill_action_details.target_h
    };
    _place_piece_on_board_js(updatedPiece, board);
    updatePieceInPiecesArray(updatedPiece);
  }
  
  function get_mutation_path_js() {
    const initial_board_state_serialized = JSON.stringify(board); // For ensuring final state is different

    for (let try_attempt = 0; try_attempt < 50; try_attempt++) { // Max attempts to find a path
        let temp_pieces = JSON.parse(JSON.stringify(pieces)); // Deep copy for planning
        let temp_board = JSON.parse(JSON.stringify(board));   // Deep copy for planning
        let current_mutation_path = [];
        let board_history_for_current_path = [JSON.stringify(temp_board)]; // Track board states for this path attempt

        const shrinkable_pieces_on_temp = get_shrinkable_pieces_js(temp_pieces);
        if (shrinkable_pieces_on_temp.length === 0) continue;

        const piece_to_shrink_on_temp = shrinkable_pieces_on_temp[Math.floor(Math.random() * shrinkable_pieces_on_temp.length)];
        const pts_orig_metrics = { x: piece_to_shrink_on_temp.x, y: piece_to_shrink_on_temp.y, w: piece_to_shrink_on_temp.width, h: piece_to_shrink_on_temp.height };
        
        const possible_shrinks_temp = get_possible_shrinks_js(piece_to_shrink_on_temp);
        if (!possible_shrinks_temp || possible_shrinks_temp.length === 0) continue;

        const chosen_shrink_details = possible_shrinks_temp[Math.floor(Math.random() * possible_shrinks_temp.length)];

        // Apply shrink on temp state
        let temp_shrunk_piece_ref = getPieceById(piece_to_shrink_on_temp.id, temp_pieces);
        _clear_piece_from_board_js(temp_shrunk_piece_ref, temp_board);
        temp_shrunk_piece_ref.x = chosen_shrink_details.new_x;
        temp_shrunk_piece_ref.y = chosen_shrink_details.new_y;
        temp_shrunk_piece_ref.width = chosen_shrink_details.new_w;
        temp_shrunk_piece_ref.height = chosen_shrink_details.new_h;
        _place_piece_on_board_js(temp_shrunk_piece_ref, temp_board);

        let current_board_serialized_after_shrink = JSON.stringify(temp_board);
        if (board_history_for_current_path.includes(current_board_serialized_after_shrink)) {
            continue; // This shrink led to an already seen state in this path, try new path
        }
        board_history_for_current_path.push(current_board_serialized_after_shrink);
        
        current_mutation_path.push({
            action_type: 'shrink', piece_id: piece_to_shrink_on_temp.id,
            original_metrics: pts_orig_metrics, applied_shrink_detail: chosen_shrink_details 
        });
        let gap_coords_temp = chosen_shrink_details.gap_coords;

        let planning_succeeded = true;
        for (let step_count = 0; step_count < 5; step_count++) { // Max fill steps
            if (is_fully_tiled_js(temp_board)) break;

            const current_gap_temp = find_empty_spaces_js(temp_board); // These are the new gap_coords
            if (current_gap_temp.length === 0) { planning_succeeded = false; break; }
            gap_coords_temp = current_gap_temp; // Update gap_coords for fill logic

            const adjacent_pieces_to_gap_temp = get_adjacent_pieces_to_gap_js(gap_coords_temp, temp_board, temp_pieces);
            let possible_fills_temp = [];

            for (const adj_piece_on_temp of adjacent_pieces_to_gap_temp) {
                const fill_actions_temp = get_fill_actions_for_piece_js(adj_piece_on_temp, gap_coords_temp);
                for (const fill_action of fill_actions_temp) {
                    let is_immediate_reversal = false;
                    if (step_count === 0 && adj_piece_on_temp.id === piece_to_shrink_on_temp.id && fill_action.action_type === "resize") {
                        if (fill_action.target_x === pts_orig_metrics.x && fill_action.target_y === pts_orig_metrics.y &&
                            fill_action.target_w === pts_orig_metrics.w && fill_action.target_h === pts_orig_metrics.h) {
                            is_immediate_reversal = true;
                        }
                    }

                    // New check for translate reversal:
                    if (!is_immediate_reversal &&
                        fill_action.action_type === "translate" &&
                        current_mutation_path.length > 0) {
                        
                        const last_action_in_path = current_mutation_path[current_mutation_path.length - 1];
                        
                        if (last_action_in_path.piece_id === adj_piece_on_temp.id &&
                            (last_action_in_path.action_type === "translate" || last_action_in_path.action_type === "resize")) {
                            // Check if the proposed translate action would move the piece back to where it was 
                            // *before* the last_action_in_path occurred.
                            if (fill_action.target_x === last_action_in_path.original_metrics_step.x &&
                                fill_action.target_y === last_action_in_path.original_metrics_step.y) {
                                is_immediate_reversal = true;
                            }
                        }
                    }

                    if (!is_immediate_reversal) {
                        possible_fills_temp.push({ piece_id: adj_piece_on_temp.id, fill_action: fill_action });
                    }
                }
            }

            if (possible_fills_temp.length === 0) { planning_succeeded = false; break; }

            // Shuffle possible_fills_temp to try them in a random order to find one that creates a new state
            for (let i = possible_fills_temp.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [possible_fills_temp[i], possible_fills_temp[j]] = [possible_fills_temp[j], possible_fills_temp[i]];
            }

            let found_valid_fill_this_step = false;
            let chosen_fill_plan_details = null; 
            let original_metrics_for_chosen_fill;

            for (const fill_option of possible_fills_temp) {
                // Simulate this fill_option
                let board_sim = JSON.parse(JSON.stringify(temp_board));
                let pieces_sim = JSON.parse(JSON.stringify(temp_pieces));
                let piece_to_fill_sim = getPieceById(fill_option.piece_id, pieces_sim);
                
                const current_piece_metrics_before_sim_fill = {
                    x: piece_to_fill_sim.x, y: piece_to_fill_sim.y,
                    w: piece_to_fill_sim.width, h: piece_to_fill_sim.height
                };

                _clear_piece_from_board_js(piece_to_fill_sim, board_sim);
                piece_to_fill_sim.x = fill_option.fill_action.target_x;
                piece_to_fill_sim.y = fill_option.fill_action.target_y;
                piece_to_fill_sim.width = fill_option.fill_action.target_w;
                piece_to_fill_sim.height = fill_option.fill_action.target_h;
                _place_piece_on_board_js(piece_to_fill_sim, board_sim);

                const board_state_after_sim_fill_serialized = JSON.stringify(board_sim);

                if (!board_history_for_current_path.includes(board_state_after_sim_fill_serialized)) {
                    // This fill is valid, select it
                    chosen_fill_plan_details = fill_option;
                    original_metrics_for_chosen_fill = current_piece_metrics_before_sim_fill;
                    
                    // Apply it to the actual temp_board and temp_pieces for the current path
                    temp_board = board_sim;
                    temp_pieces = pieces_sim;
                    board_history_for_current_path.push(board_state_after_sim_fill_serialized);
                    found_valid_fill_this_step = true;
                    break; 
                }
            }

            if (!found_valid_fill_this_step) {
                planning_succeeded = false; break;
            }
            
            // chosen_fill_plan is now chosen_fill_plan_details
            // original_metrics_for_this_step is now original_metrics_for_chosen_fill
            current_mutation_path.push({
                action_type: chosen_fill_plan_details.fill_action.action_type, 
                piece_id: chosen_fill_plan_details.piece_id,
                original_metrics_step: original_metrics_for_chosen_fill, 
                new_pos: { x: chosen_fill_plan_details.fill_action.target_x, y: chosen_fill_plan_details.fill_action.target_y },
                new_size: { w: chosen_fill_plan_details.fill_action.target_w, h: chosen_fill_plan_details.fill_action.target_h }
            });
        }

        if (planning_succeeded && is_fully_tiled_js(temp_board)) {
            const final_path_board_state_serialized = JSON.stringify(temp_board);
            if (final_path_board_state_serialized !== initial_board_state_serialized) {
                return current_mutation_path;
            }
        }
    }
    return []; // No path found
  }
  
  let animatingPieceIds = new Set();

  async function apply_mutation_path_to_display(path) {
    if (!path || path.length === 0) return;

    const currentAnimatingIds = new Set(path.map(action => action.piece_id));
    currentAnimatingIds.forEach(id => animatingPieceIds.add(id));
    animatingPieceIds = new Set(animatingPieceIds); // Trigger Svelte update for class

    for (let i = 0; i < path.length; i++) {
        const action = path[i];
        // Apply action to actual state (global `pieces` and `board`)
        if (action.action_type === 'shrink') {
            apply_shrink_js_to_state(action.piece_id, action.applied_shrink_detail);
        } else if (action.action_type === 'translate' || action.action_type === 'resize') {
            apply_fill_action_js_to_state(action.piece_id, {
                target_x: action.new_pos.x, target_y: action.new_pos.y,
                target_w: action.new_size.w, target_h: action.new_size.h
            });
        }
        if (i < path.length -1) { // Add delay between steps if not the last step
             await new Promise(resolve => setTimeout(resolve, 1000)); // Stagger subsequent steps
        }
    }
    
    setTimeout(() => {
        currentAnimatingIds.forEach(id => animatingPieceIds.delete(id));
        animatingPieceIds = new Set(animatingPieceIds); // Trigger Svelte update
    }, 800); // Corresponds to CSS transition time
  }

  function performMutation() {
    if (!is_fully_tiled_js(board)) {
        console.log("Grid not fully tiled, attempting to regenerate or skip mutation.");
        generateInitialGrid(); // Attempt to fix if grid is broken
        return;
    }
    const path = get_mutation_path_js();
    if (path && path.length > 0) {
      apply_mutation_path_to_display(path);
    }
  }

  onMount(() => {
    generateInitialGrid();
    const intervalId = setInterval(performMutation, MUTATION_INTERVAL);
    return () => clearInterval(intervalId);
  });

</script>

<style>
  :global(body) {
    background: #0a0a0a;
    font-family: 'Arial', sans-serif;
    padding: 0; 
    min-height: 100vh;
    margin: 0; 
    box-sizing: border-box;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  :global(*, *:before, *:after) {
    box-sizing: inherit;
  }

  .grid-container {
    display: grid;
    grid-template-columns: repeat(var(--grid-width, 8), minmax(60px, 1fr)); /* Ensure min size */
    grid-auto-rows: minmax(100px, auto); 
    gap: 12px;
    width: 90vw; /* Responsive width */
    max-width: 1200px;
    margin: 20px auto; 
    padding: 10px; 
    background-color: #1a1a1a30; /* Slightly transparent */
    border-radius: 8px;
    border: 1px solid #ffffff10;
  }

  .card {
    position: relative;
    cursor: pointer;
    transform-style: preserve-3d;
    perspective: 1000px;
    border-radius: 12px;
    background-color: #0f0f0f; 
    transition: grid-column-start 0.8s cubic-bezier(0.4, 0, 0.2, 1),
                grid-row-start 0.8s cubic-bezier(0.4, 0, 0.2, 1),
                grid-column-end 0.8s cubic-bezier(0.4, 0, 0.2, 1),
                grid-row-end 0.8s cubic-bezier(0.4, 0, 0.2, 1);
    filter: drop-shadow(0 0 8px rgba(255, 0, 110, 0.25)) 
            drop-shadow(0 0 15px rgba(131, 56, 236, 0.15));
  }

  /* .card.animating could be used for more specific transition states if needed */

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
    flex-direction: column; 
    align-items: center;
    justify-content: center;
    padding: 15px; 
    overflow: hidden; 
  }

  .card-front {
    border: 3px solid;
    border-image: linear-gradient(45deg, #ff006e, #8338ec, #3a86ff, #06ffa5) 1;
    border-image-slice: 1;
  }

  .card-front::before { /* Gradient Border Glow */
    content: '';
    position: absolute;
    top: -3px; left: -3px; right: -3px; bottom: -3px;
    background: linear-gradient(45deg, #ff006e, #8338ec, #3a86ff, #06ffa5);
    background-size: 400% 400%;
    border-radius: 12px; 
    z-index: -2; /* Behind border-image */
    animation: gradientShift 4s ease infinite;
  }

  .card-front::after { /* Inner Background */
    content: '';
    position: absolute;
    top: 0px; left: 0px; right: 0px; bottom: 0px; /* Cover full area inside border */
    background: #0a0a0a; 
    border-radius: 9px; 
    z-index: -1; /* Behind content, above border glow */
  }
  
  .card-front-content { 
    color: white;
    text-align: center;
    z-index: 1; /* Above ::after */
    font-size: 0.7em;
    opacity: 0.5;
  }

  .card-back {
    background: #1e1e1e; 
    color: #eee; 
    transform: rotateY(180deg);
    text-align: center;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.5);
  }

  .card-back h3 {
    font-size: 1em; 
    margin-bottom: 8px;
    color: #fafafa; 
  }

  .card-back p {
    font-size: 0.75em; 
    line-height: 1.4;
    color: #ccc; 
  }

  @keyframes gradientShift {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
  }

  @media (max-width: 768px) {
    .grid-container {
      grid-template-columns: repeat(var(--grid-width-tablet, 4), 1fr);
      grid-auto-rows: minmax(80px, auto);
      gap: 10px;
    }
    /* Note: GRID_WIDTH itself is JS controlled. For responsive column count changes,
       JS would need to detect viewport and regenerate grid with new GRID_WIDTH.
       The --grid-width-tablet CSS var is illustrative if CSS were to control columns. */
    .card-back h3 { font-size: 0.9em; }
    .card-back p { font-size: 0.7em; }
  }

  @media (max-width: 480px) {
    .grid-container {
      grid-template-columns: repeat(var(--grid-width-mobile, 2), 1fr);
      grid-auto-rows: minmax(70px, auto);
      gap: 8px;
    }
     .card-front, .card-back { padding: 10px; }
     .card-back h3 { font-size: 0.8em; }
     .card-back p { font-size: 0.65em; }
  }
</style>

<div class="grid-container" style="--grid-width: {GRID_WIDTH};">
  {#each pieces as piece (piece.key)}
    <div 
      class="card" 
      class:animating={animatingPieceIds.has(piece.id)}
      style="
        grid-column-start: {piece.x + 1}; 
        grid-row-start: {piece.y + 1}; 
        grid-column-end: span {piece.width}; 
        grid-row-end: span {piece.height};
      "
    >
      <div class="card-inner">
        <div class="card-front">
           <div class="card-front-content">
             ID: {piece.id}
           </div>
        </div>
        <div class="card-back">
          <h3>{piece.title}</h3>
          <p>{piece.text}</p>
        </div>
      </div>
    </div>
  {/each}
</div>
