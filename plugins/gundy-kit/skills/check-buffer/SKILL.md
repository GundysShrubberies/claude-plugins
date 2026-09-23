---
name: check-buffer
description: Review the file currently open in the user's nvim buffer. Use when the user says "check my buffer", "review what I'm working on", "look at my current file", or otherwise refers to their open editor file without naming a path.
---

# Check buffer

Reviews whatever the user has open in nvim right now, so they can ask for a review
without breaking flow to type a path.

## How the path arrives

This depends on an autocmd in your nvim config, `init.lua`, that writes the active
buffer's absolute path to `/tmp/nvim_current_buffer` on `BufEnter`/`BufWritePost`.
Without it, this skill has nothing to read. Add it once:

```lua
-- Track current buffer for Claude context
vim.api.nvim_create_autocmd({ "BufEnter", "BufWritePost" }, {
  callback = function()
    local path = vim.fn.expand('%:p')
    if path ~= '' then
      local f = io.open('/tmp/nvim_current_buffer', 'w')
      if f then f:write(path) f:close() end
    end
  end,
})
```

That file holds one line: the path of whatever buffer nvim most recently entered
or saved.

## Steps

1. Read `/tmp/nvim_current_buffer`.
   - Missing or empty → nvim isn't running, or hasn't entered a buffer since it
     started (or the autocmd above isn't installed). Say so and ask which file they
     mean. Don't guess.
2. Read the path it contains.
   - Path doesn't resolve → the buffer is unsaved or was deleted. Say which, and stop.
3. Review the file. Prioritize in this order:
   - **Correctness** — logic errors, unhandled cases, wrong assumptions.
   - **Fit** — does it do what the surrounding code and any active task expect?
   - **Clarity** — only where it genuinely impedes reading.
4. If the user is working through a lesson plan or task, check the file against it
   and say explicitly whether it satisfies the objective.

## Output

Lead with the verdict, then the findings, most important first. Reference lines as
`path:line`. If nothing is wrong, say that in one line rather than manufacturing
nits — a clean review is a useful result.
