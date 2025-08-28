# input_handler

A tiny, reusable input editor for Rust CLI apps. It provides readline-like terminal editing with in-memory and optional on-disk history, arrow-key navigation, backspace/delete handling, and sane defaults, without external C deps beyond `libc`/`termios`.

## Features
- **Line editing**: insert, backspace, left/right cursor movement
- **History navigation**: up/down to traverse previous inputs
- **Persistent history (optional)**: load/save to a file you choose
- **Minimal API**: `Editor` and ergonomic wrapper `InputHandler`
- **Raw mode management**: enables raw mode and restores it on drop

## Installation
Add to your `Cargo.toml`:
```toml
[dependencies]
input_handler = "0.1"
```

## Quick start
```rust
use input_handler::InputHandler;

fn main() -> std::io::Result<()> {
	let mut ih = InputHandler::new()?;
	loop {
		let line = ih.readline("app> ")?;
		if line.trim() == "exit" { break; }
		println!("you typed: {}", line);
	}
	Ok(())
}
```

### With persistent history
```rust
use input_handler::InputHandler;
use std::path::PathBuf;

fn main() -> std::io::Result<()> {
	let history_path = PathBuf::from("/tmp/.app_history");
	let mut ih = InputHandler::with_history_file(history_path)?;
	let line = ih.readline("app> ")?;
	println!("{}", line);
	Ok(())
}
```

## Key bindings
- **Enter**: submit current line
- **Backspace**: delete character before cursor
- **Left/Right**: move cursor
- **Up/Down**: previous/next history entry
- **Ctrl-C**: returns an empty string (line is cleared), prints `^C`
- **Ctrl-D**: when the buffer is empty, returns the literal string `"exit"`

## API overview
Exports from the crate root:
- `Editor`
  - `new() -> io::Result<Editor>`
  - `with_history_file(path: PathBuf) -> io::Result<Editor>`
  - `readline(prompt: &str) -> io::Result<String>`
- `History`
  - `new(max_size: usize) -> History`
  - `with_file(max_size: usize, path: PathBuf) -> History`
  - `add(command: String)`
  - `previous() -> Option<&String>` / `next_command() -> Option<&String>`
- `InputHandler` (light wrapper around `Editor`)
  - `new() -> io::Result<InputHandler>`
  - `with_history_file(path: PathBuf) -> io::Result<InputHandler>`
  - `readline(prompt: &str) -> io::Result<String>`

Notes:
- History de-duplicates consecutive identical inputs and ignores empty lines.
- When backed by a file, history is loaded on start and rewritten on each add.
- Raw mode is enabled before reading and restored automatically on drop.

## Platform support
This crate uses `termios` and `libc` to enter raw mode and read keys. It targets Unix-like systems (Linux, macOS). Windows is not supported out of the box.

## Example behavior
- Pressing Up/Down cycles through history. When you reach the newest entry and press Down again, the buffer clears.
- Pressing Ctrl-D with an empty buffer returns `"exit"` so you can exit loops easily; otherwise, it is ignored.

## License
MIT © Contributors

## Repository
`https://github.com/Arshdeep54/input_handler`