# Fedora AI/ML Local Setup

A beginner-friendly record of my local AI/ML environment and my contributions to the Fedora AI/ML SIG.

## Hardware

- Laptop: Lenovo ThinkPad T14 Gen 3
- CPU: Intel Core i7-1265U (12 logical processors)
- Memory: 32 GiB RAM
- Graphics: integrated Intel Iris Xe
- Display server: Wayland
- Fedora: Fedora Linux 44 KDE Plasma
- KDE Plasma: 6.8.2
- Kernel: 7.2.15-200.fc44.x86_64

This setup is CPU-only. Larger models work, but response speed depends heavily on the task and context size.

## Ollama

[Ollama](https://ollama.com/) runs language models locally.

Check installed models:

```bash
ollama list
```

Check which models are currently loaded:

```bash
ollama ps
```

Run a model:

```bash
ollama run qwen3-8b-10t
```

The `-10t` suffix identifies my custom Ollama model configuration using 10 CPU threads. It does not mean the model has ten billion parameters.

A model can be kept in memory for continued work:

```bash
curl -s http://127.0.0.1:11434/api/generate \
  -d '{"model":"qwen3-8b-10t","keep_alive":-1}' > /dev/null
```

Verify:

```bash
ollama ps
```

To stop it and release memory:

```bash
ollama stop qwen3-8b-10t
```

## Models tested

- DeepSeek-R1 7B: usable, but slower on CPU
- DeepSeek-R1 14B: better quality, considerably slower
- Qwen3 8B: faster and suitable for local agent experiments
- Qwen3 14B: more capable, but slower than Qwen3 8B

For practical work on this laptop, Qwen3 8B is currently the best balance between speed and capability. Model choice can change as hardware improves.

## Pi coding agent

Pi is the agent framework. It connects a model provider such as Ollama to tools, files, skills, and a working directory.

Example:

```bash
pi --model ollama/qwen3-8b-10t:latest
```

A Pi session can be resumed with:

```bash
pi --session SESSION_ID
```

Pi provides tools such as reading and writing files. The model itself generates the response, while Pi performs the tool calls and reports the results.

## Fedora Agent Skills

The Fedora AI/ML SIG publishes reusable Agent Skills. I installed the skills COPR and verified that Pi can discover these skills:

- fedora-compose-triage
- fedora-ftbfs-search
- fedora-packaging
- meeting-secretary
- packit-pull-from-upstream-onboarding

The skills are available under:

```text
/usr/share/agents/skills/
```

The `meeting-secretary` skill was tested locally. It extracts:

- The team
- Major themes
- Links
- Action items

## Meeting-secretary test

A large Fedora AI/ML SIG Meetbot HTML log was approximately 86 KB. Processing the full log with a local Qwen 8B model and an 8K context window timed out.

To validate the workflow, I created a smaller local test file containing a condensed meeting summary. Pi successfully:

1. Loaded the `meeting-secretary` skill
2. Read the local HTML file
3. Used the read tool
4. Extracted the team, themes, links, and action items

This confirmed the skill workflow, while also showing that low-context local models need smaller inputs or preprocessing.

## Tool-use validation

I tested whether Pi and the local model could perform sequential file operations:

1. Write a file named `result.txt`
2. Read the file afterward
3. Report the number of lines

The file contained exactly three lines, and the model correctly reported three lines.

Timing examples on the laptop:

- Qwen3 8B warm run: about 1 minute 14 seconds
- Qwen3 8B timed run: about 2 minutes 41 seconds
- Qwen3 14B warm run: about 3 minutes 19 seconds
- Qwen3 14B cold run: about 4 minutes 5 seconds

Warm runs are faster because the model is already loaded in memory.

## Fedora TWIF WordPress export skill

I also tested Justin Wheeler's Fedora AI/ML SIG skill for converting a “This Week in Fedora” Markdown edition into WordPress Gutenberg-compatible HTML.

The local workflow was:

```bash
cd ~/skills-library

pytest -q

python3 skills/fedora-twif-wordpress-export/assets/twif_wordpress.py fetch \
  --article content/2026/11-02.md \
  --ref main \
  --output-dir /tmp/twif-run

python3 skills/fedora-twif-wordpress-export/assets/twif_wordpress.py render \
  --source-bundle /tmp/twif-run/source-manifest.json \
  --start-date 2026-11-02 \
  --end-date 2026-11-08 \
  --output-dir /tmp/twif-run
```

Results:

- 70 offline tests passed
- The Forge fetch produced a source manifest
- The render step produced Gutenberg HTML, a preview, and a conversion report
- The generated article contained 366 blocks and 32 wrapped sections
- No detailed reports were available for that edition
- The generated output did not yet contain an AI disclosure, matching an issue noted in the pull request

The Forge fetch required a personal read-only Forge token with both repository read access and user read access. Tokens must never be committed to this repository.

## Open WebUI

Open WebUI was run locally with Podman to provide a browser interface for Ollama:

```bash
podman start openwebui
```

It was available at:

```text
http://127.0.0.1:3000
```

The terminal interface was generally faster for this CPU-only setup. Open WebUI is more convenient for file uploads and conversations, but it adds some interface overhead.

## Lessons learned

- Local models can be very useful without a dedicated GPU.
- Smaller context windows require smaller inputs or preprocessing.
- Keeping a model warm in RAM improves response time.
- Pi is the framework; Ollama supplies the local model; Agent Skills provide reusable task instructions.
- A successful unit test does not guarantee that a full real-world workflow will work, so both offline tests and a safe local integration test are useful.
- Public documentation should explain limitations, timing, and failed attempts as well as successful commands.

## Next steps

- Improve the meeting-log preprocessing workflow for 8K context windows.
- Add screenshots and clearer troubleshooting notes where useful.
- Test additional Fedora Agent Skills with small, read-only inputs.
- Turn this setup record into a beginner-friendly Fedora guide.
- Continue peer-reviewing Fedora AI/ML SIG skills.
