# Tavus Skills

Agent skills for building with [Tavus](https://tavus.io) - real-time conversational video AI and video generation.

## Install

```bash
# Install all skills
npx skills add tavus-io/skills

# Install specific skills
npx skills add tavus-io/skills --skill tavus-cvi-quickstart
npx skills add tavus-io/skills --skill tavus-video-gen
```

## Available Skills

### Overview
| Skill | Description |
|-------|-------------|
| **tavus-overview** | What is Tavus? Human Computing, CVI concepts, model stack (Phoenix/Raven/Sparrow), platform links, pricing, and company info. |

### Video Generation
| Skill | Description |
|-------|-------------|
| **tavus-replica** | Create and manage AI digital twins (replicas). Training videos, consent, Phoenix-3 model. |
| **tavus-video-gen** | Generate personalized AI videos from scripts or audio. Backgrounds, watermarks, batch generation. |

### Conversational Video Interface (CVI)
| Skill | Description |
|-------|-------------|
| **tavus-cvi-quickstart** | Get started with real-time AI video conversations. Stock replicas/personas, first conversation. |
| **tavus-cvi-persona** | Deep persona configuration. Custom LLMs, TTS engines, perception (Raven), turn-taking (Sparrow). |
| **tavus-cvi-interactions** | Control conversations in real-time. Echo text, interrupt, inject context, handle events via WebRTC. |
| **tavus-cvi-ui** | React components for embedding CVI. @tavus/cvi-ui, hooks, Vite/Next.js integration. |
| **tavus-cvi-knowledge** | Knowledge base (RAG) and memories. Upload documents, persistent context across conversations. |

## Quick Links

- [Tavus Documentation](https://docs.tavus.io)
- [API Reference](https://docs.tavus.io/api-reference/overview)
- [Developer Portal](https://platform.tavus.io)
- [Example Projects](https://github.com/Tavus-Engineering/tavus-examples)

## Get an API Key

Sign up at [platform.tavus.io](https://platform.tavus.io/auth/sign-up?is_developer=true)

## License

MIT
