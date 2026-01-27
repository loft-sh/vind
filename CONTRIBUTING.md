# Contributing to vind

Thank you for your interest in contributing to vind! This document provides guidelines and instructions for contributing.

## What is vind?

vind (vCluster in Docker) is a documentation and examples repository for the vCluster Docker driver feature. The core code lives in the [vCluster repository](https://github.com/loft-sh/vcluster).

## How to Contribute

### Documentation Improvements

We welcome contributions to improve documentation:

- Fix typos and grammar
- Clarify confusing sections
- Add missing information
- Improve examples
- Add new use cases

### Examples

Share your vind configurations and use cases:

- Real-world examples
- Best practices
- Common patterns
- Troubleshooting solutions

### Reporting Issues

If you find issues with vind or the Docker driver:

1. Check if it's a vCluster issue: [vCluster Issues](https://github.com/loft-sh/vcluster/issues)
2. Check existing issues in this repository
3. Create a new issue with:
   - Clear description
   - Steps to reproduce
   - Expected vs actual behavior
   - Environment details

## Contribution Process

1. **Fork the repository**
   ```bash
   git clone https://github.com/saiyam1814/vind.git
   cd vind
   ```

2. **Create a branch**
   ```bash
   git checkout -b my-contribution
   ```

3. **Make your changes**
   - Edit documentation
   - Add examples
   - Fix issues

4. **Test your changes**
   - Verify examples work
   - Check markdown formatting
   - Test commands

5. **Commit your changes**
   ```bash
   git add .
   git commit -m "Description of your changes"
   ```

6. **Push and create PR**
   ```bash
   git push origin my-contribution
   ```
   Then create a pull request on GitHub.

## Documentation Style

### Markdown Guidelines

- Use clear headings
- Include code examples
- Add usage instructions
- Link to related docs

### Code Examples

- Always include usage instructions
- Use realistic examples
- Test examples before submitting
- Include expected output

### File Structure

- Keep files organized
- Use descriptive names
- Follow existing structure
- Update README if adding new sections

## Example Contribution

### Adding a New Example

1. Create example file in `examples/`:
   ```yaml
   # examples/my-example.yaml
   # Description of what this example does
   
   experimental:
     docker:
       # Configuration
   ```

2. Update `examples/README.md`:
   ```markdown
   ### [my-example.yaml](./my-example.yaml)
   Description and usage instructions.
   ```

3. Test the example:
   ```bash
   vcluster create test-cluster -f examples/my-example.yaml
   ```

4. Submit PR with clear description.

## Code Contributions

For code changes to the Docker driver itself, please contribute to:
- [vCluster Repository](https://github.com/loft-sh/vcluster)
- Follow [vCluster Contributing Guide](https://github.com/loft-sh/vcluster/blob/main/CONTRIBUTING.md)

## Questions?

- 💬 [Slack Community](https://slack.loft.sh/)
- 🐛 [GitHub Issues](https://github.com/saiyam1814/vind/issues)
- 💡 [Discussions](https://github.com/saiyam1814/vind/discussions)

## Code of Conduct

Please be respectful and constructive in all interactions. We follow the [Contributor Covenant Code of Conduct](https://www.contributor-covenant.org/).

Thank you for contributing to vind! 🎉
