# Contributing to iOS Interview Preparation Guide

First off, thank you for considering contributing to the iOS Interview Preparation Guide! 🎉

This guide has helped hundreds of developers land their dream iOS positions, and your contributions make it even better.

## 🌟 How Can I Contribute?

### 1. **Report Bugs or Issues**
- Found an error in code examples?
- Spotted a typo or grammatical mistake?
- Is something unclear or confusing?

**Please open an issue** with:
- Clear description of the problem
- Location (file and section)
- Suggested fix (if applicable)

### 2. **Suggest New Interview Questions**
Have you been asked an interesting iOS interview question? Share it!

**Include:**
- The question
- Your detailed answer
- Code example (if applicable)
- Company name (optional)
- Difficulty level (entry/mid/senior)

### 3. **Improve Existing Content**
- Enhance code examples
- Add better explanations
- Include more real-world scenarios
- Update for latest iOS versions
- Add diagrams or visual aids

### 4. **Add New Topics**
- Advanced features not yet covered
- Emerging iOS technologies
- Best practices and patterns
- Performance optimization techniques

## 📝 Contribution Guidelines

### Code Examples

All Swift code should:
- ✅ Follow Swift API Design Guidelines
- ✅ Use Swift 5.9+ syntax
- ✅ Be production-ready and tested
- ✅ Include inline comments for complex logic
- ✅ Use meaningful variable names
- ✅ Follow iOS best practices

**Example:**
```swift
// ✅ Good
class UserViewModel {
    weak var delegate: UserViewModelDelegate?
    
    func fetchUser(id: String) async throws -> User {
        let user = try await networkService.getUser(id: id)
        return user
    }
}

// ❌ Avoid
class vm {
    var d: Any?
    func fetch(_ i: String) { }
}
```

### Documentation Style

- Use clear, concise language
- Write for non-native English speakers
- Include practical examples
- Format code blocks with syntax highlighting
- Add links to official Apple documentation when relevant

### File Structure

When adding new content:
```
docs/
├── your-new-topic.md       # Main content
└── QUESTIONS_INDEX.md      # Update with new questions
```

## 🔄 Pull Request Process

### 1. Fork the Repository
Click the "Fork" button at the top right of the repository page.

### 2. Clone Your Fork
```bash
git clone https://github.com/YOUR-USERNAME/ios-interview-prep.git
cd ios-interview-prep
```

### 3. Create a Branch
```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/your-fix-name
```

**Branch naming:**
- `feature/` - New features or content
- `fix/` - Bug fixes or corrections
- `docs/` - Documentation improvements
- `update/` - Content updates

### 4. Make Your Changes
- Follow the guidelines above
- Test code examples
- Proofread content
- Check markdown formatting

### 5. Commit Your Changes
```bash
git add .
git commit -m "Add: Description of your changes"
```

**Commit message format:**
- `Add: ` - New content
- `Fix: ` - Bug fixes
- `Update: ` - Content updates
- `Improve: ` - Enhancements
- `Docs: ` - Documentation changes

**Examples:**
- ✅ `Add: SwiftUI data flow patterns to advanced topics`
- ✅ `Fix: Typo in memory management section`
- ✅ `Update: Concurrency examples for Swift 6`
- ✅ `Improve: Code examples in networking section`

### 6. Push to Your Fork
```bash
git push origin feature/your-feature-name
```

### 7. Create a Pull Request
- Go to your fork on GitHub
- Click "New Pull Request"
- Provide a clear title and description
- Reference any related issues

**PR Description Template:**
```markdown
## What does this PR do?
Brief description of changes

## Type of Change
- [ ] New content/feature
- [ ] Bug fix
- [ ] Content improvement
- [ ] Documentation update

## Checklist
- [ ] Code examples tested
- [ ] Markdown properly formatted
- [ ] No typos or grammatical errors
- [ ] Updated QUESTIONS_INDEX.md (if applicable)
- [ ] Follows contribution guidelines

## Related Issue
Closes #issue_number (if applicable)
```

### 8. Review Process
- Maintainers will review your PR
- Address any feedback or requested changes
- Once approved, your PR will be merged!

## 🎯 Content Quality Standards

### Interview Questions

Each question should include:
1. **Clear Question** - Exactly as asked in interviews
2. **Detailed Answer** - Production-quality explanation
3. **Code Example** - Working Swift code (when applicable)
4. **Follow-up Points** - Additional discussion topics
5. **Real-world Use Case** - Practical application

**Example Structure:**
```markdown
### Q: What's the difference between weak and unowned references?

**Answer:**

Both `weak` and `unowned` references are used to prevent retain cycles in Swift, but they differ in important ways:

**Weak References:**
- Always optional (`weak var delegate: SomeDelegate?`)
- Automatically set to `nil` when deallocated
- Use when referenced object may be deallocated first
- Safer option

**Code Example:**
[Include working Swift code]

**When to Use:**
[Practical guidelines]

**Interview Follow-ups:**
- How do retain cycles occur?
- When would you use unowned instead?
```

### Code Quality

All code must:
- ✅ Compile without errors
- ✅ Follow Swift conventions
- ✅ Use modern Swift syntax
- ✅ Include error handling
- ✅ Be production-ready
- ✅ Have clear comments

## 🚀 Quick Contribution Ideas

### Easy Contributions (Great for First-Timers!)
- Fix typos or grammatical errors
- Improve code formatting
- Add comments to code examples
- Update outdated links
- Improve markdown formatting

### Medium Contributions
- Add new code examples
- Expand existing explanations
- Add new interview questions
- Create diagrams or visual aids
- Update content for latest iOS version

### Advanced Contributions
- Add new comprehensive topics
- Create architectural diagrams
- Add performance comparison benchmarks
- Develop interactive examples
- Create video explanations (linked content)

## 📋 Content Checklist

Before submitting, ensure:
- [ ] Content is technically accurate
- [ ] Code examples are tested and work
- [ ] Markdown is properly formatted
- [ ] No spelling or grammar errors
- [ ] Follows repository structure
- [ ] QUESTIONS_INDEX.md is updated (if adding questions)
- [ ] Links are working
- [ ] Content is interview-focused

## 💡 Style Guide

### Markdown Formatting
- Use `#` for main titles
- Use `##` for sections
- Use `###` for subsections
- Use code blocks with language specification: ```swift
- Use **bold** for emphasis
- Use *italic* for terms
- Use `inline code` for code references

### Writing Style
- Write in present tense
- Use active voice
- Be concise but comprehensive
- Include practical examples
- Explain "why" not just "what"
- Consider international audience

## 🏆 Recognition

Contributors will be:
- Listed in repository contributors
- Mentioned in release notes (for significant contributions)
- Part of a community helping developers worldwide

## 📞 Need Help?

- **Questions?** Open a discussion
- **Unclear guidelines?** Ask in issues
- **Want to contribute but unsure how?** We'll guide you!

## 📜 Code of Conduct

### Our Standards

- ✅ Be respectful and inclusive
- ✅ Welcome newcomers
- ✅ Accept constructive criticism
- ✅ Focus on what's best for the community
- ✅ Show empathy towards others

### Unacceptable Behavior

- ❌ Harassment or discrimination
- ❌ Trolling or insulting comments
- ❌ Personal or political attacks
- ❌ Publishing others' private information
- ❌ Unprofessional conduct

## 📊 What Happens Next?

1. **Submit PR** → Your contribution is reviewed
2. **Feedback** → Address any requested changes
3. **Approval** → PR is approved by maintainer
4. **Merge** → Your contribution is merged!
5. **Deploy** → Changes go live on GitHub Pages
6. **Impact** → Help developers worldwide! 🌍

## 🎉 Thank You!

Every contribution, no matter how small, makes a difference. Thank you for helping make iOS interview preparation accessible to developers worldwide!

---

**Questions?** Feel free to reach out by opening an issue or discussion.

**Ready to contribute?** Check out our [open issues](https://github.com/mukundjogi/ios-interview-prep/issues) or [start a discussion](https://github.com/mukundjogi/ios-interview-prep/discussions)!

Happy Contributing! 🚀🍎

