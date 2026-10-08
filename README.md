# teamforward-agent-site

Claude Code skill for building and launching a Team Forward agent website.

## Install

```bash
/plugin marketplace add KCDryan/teamforward-agent-site
/plugin install teamforward-agent-site@kcd-skills
```

Then say "build a Team Forward site for [agent name]" or run `/teamforward-agent-site`.

## Requirements

- Git and GitHub access. On first run the skill clones the template from the public repo
  [KCDryan/kirbychan-toronto](https://github.com/KCDryan/kirbychan-toronto) into `~/kirbychan-toronto`.
- A Cloudflare account for deploy. Owner secrets are set in Cloudflare, never in chat.

## License

MIT. See [LICENSE](LICENSE).
