# iOS UI Development (UIKit)

[← Back to Main](../README.md) | [Previous: OOP and POP](oop-and-pop.md) | [Next: SwiftUI →](swiftui.md)

## Table of Contents
- [Storyboards vs XIB vs Programmatic UI](#storyboards-vs-xib-vs-programmatic-ui)
- [Auto Layout & Constraints](#auto-layout--constraints)
- [UIKit Navigation](#uikit-navigation)
- [TableView & CollectionView](#tableview--collectionview)
- [Gesture Recognizers & Touch Events](#gesture-recognizers--touch-events)
- [Accessibility](#accessibility-in-ios-apps)

---

## Storyboards vs XIB vs Programmatic UI

### Storyboards

Visual interface files that show all view controllers and their relationships.

**Advantages:**
- Visual representation of app flow
- Easy to see navigation paths
- Segues for transitions
- Prototype cells for TableView/CollectionView
- Good for beginners

**Disadvantages:**
- Merge conflicts in teams
- Slower to load for large apps
- Hard to reuse components
- Version control issues
- Can become unwieldy

**Example:**

```swift
class StoryboardViewController: UIViewController {
    // IBOutlet connections from storyboard
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var submitButton: UIButton!
    @IBOutlet weak var textField: UITextField!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        titleLabel.text = "Welcome"
        submitButton.layer.cornerRadius = 8
    }
    
    // IBAction from button
    @IBAction func submitTapped(_ sender: UIButton) {
        guard let text = textField.text, !text.isEmpty else {
            showAlert(message: "Please enter text")
            return
        }
        processText(text)
    }
    
    // Segue
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if segue.identifier == "showDetail" {
            if let detailVC = segue.destination as? DetailViewController {
                detailVC.data = textField.text
            }
        }
    }
    
    private func showAlert(message: String) {
        let alert = UIAlertController(title: "Error", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
    
    private func processText(_ text: String) {
        print("Processing: \(text)")
    }
}
```

### XIB Files

Individual interface files for single views or view controllers.

**Advantages:**
- Reusable across different view controllers
- Better for custom cells and views
- Less merge conflicts than storyboards
- Smaller file size

**Disadvantages:**
- No visual navigation flow
- More files to manage
- Still have merge conflict potential

**Example:**

```swift
class CustomTableViewCell: UITableViewCell {
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var subtitleLabel: UILabel!
    @IBOutlet weak var iconImageView: UIImageView!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        setupUI()
    }
    
    private func setupUI() {
        iconImageView.layer.cornerRadius = iconImageView.frame.width / 2
        iconImageView.clipsToBounds = true
    }
    
    func configure(with item: Item) {
        titleLabel.text = item.title
        subtitleLabel.text = item.subtitle
        iconImageView.image = UIImage(named: item.iconName)
    }
}

// Loading XIB in code
class CustomView: UIView {
    @IBOutlet var contentView: UIView!
    @IBOutlet weak var label: UILabel!
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        commonInit()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        commonInit()
    }
    
    private func commonInit() {
        Bundle.main.loadNibNamed("CustomView", owner: self, options: nil)
        addSubview(contentView)
        contentView.frame = self.bounds
        contentView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
    }
}

struct Item {
    let title: String
    let subtitle: String
    let iconName: String
}
```

### Programmatic UI

Creating UI entirely in code.

**Advantages:**
- No merge conflicts
- Full control and flexibility
- Easy to reuse and test
- Better for dynamic UIs
- Easier code review
- Strongly typed

**Disadvantages:**
- Steeper learning curve
- More code to write
- No visual preview (unless using SwiftUI preview)

**Example:**

```swift
class ProgrammaticViewController: UIViewController {
    
    // MARK: - UI Components
    
    private let titleLabel: UILabel = {
        let label = UILabel()
        label.text = "Welcome"
        label.font = .systemFont(ofSize: 24, weight: .bold)
        label.textAlignment = .center
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()
    
    private let descriptionLabel: UILabel = {
        let label = UILabel()
        label.text = "Enter your details below"
        label.font = .systemFont(ofSize: 16)
        label.textColor = .gray
        label.textAlignment = .center
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()
    
    private let textField: UITextField = {
        let textField = UITextField()
        textField.placeholder = "Enter text"
        textField.borderStyle = .roundedRect
        textField.translatesAutoresizingMaskIntoConstraints = false
        return textField
    }()
    
    private let submitButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("Submit", for: .normal)
        button.setTitleColor(.white, for: .normal)
        button.backgroundColor = .systemBlue
        button.layer.cornerRadius = 8
        button.translatesAutoresizingMaskIntoConstraints = false
        return button
    }()
    
    // MARK: - Lifecycle
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        setupConstraints()
        setupActions()
    }
    
    // MARK: - Setup
    
    private func setupUI() {
        view.backgroundColor = .white
        view.addSubview(titleLabel)
        view.addSubview(descriptionLabel)
        view.addSubview(textField)
        view.addSubview(submitButton)
    }
    
    private func setupConstraints() {
        NSLayoutConstraint.activate([
            // Title Label
            titleLabel.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
            titleLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            titleLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            
            // Description Label
            descriptionLabel.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 8),
            descriptionLabel.leadingAnchor.constraint(equalTo: titleLabel.leadingAnchor),
            descriptionLabel.trailingAnchor.constraint(equalTo: titleLabel.trailingAnchor),
            
            // Text Field
            textField.topAnchor.constraint(equalTo: descriptionLabel.bottomAnchor, constant: 40),
            textField.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            textField.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            textField.heightAnchor.constraint(equalToConstant: 44),
            
            // Submit Button
            submitButton.topAnchor.constraint(equalTo: textField.bottomAnchor, constant: 20),
            submitButton.leadingAnchor.constraint(equalTo: textField.leadingAnchor),
            submitButton.trailingAnchor.constraint(equalTo: textField.trailingAnchor),
            submitButton.heightAnchor.constraint(equalToConstant: 50)
        ])
    }
    
    private func setupActions() {
        submitButton.addTarget(self, action: #selector(submitTapped), for: .touchUpInside)
    }
    
    // MARK: - Actions
    
    @objc private func submitTapped() {
        guard let text = textField.text, !text.isEmpty else {
            showAlert(message: "Please enter text")
            return
        }
        
        let detailVC = DetailViewController()
        detailVC.data = text
        navigationController?.pushViewController(detailVC, animated: true)
    }
    
    private func showAlert(message: String) {
        let alert = UIAlertController(title: "Error", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

### Comparison

| Feature | Storyboard | XIB | Programmatic |
|---------|-----------|-----|--------------|
| **Learning Curve** | Easy | Medium | Hard |
| **Team Collaboration** | Difficult | Medium | Easy |
| **Merge Conflicts** | High | Medium | None |
| **Reusability** | Low | Medium | High |
| **Dynamic UI** | Limited | Limited | Excellent |
| **Performance** | Slower load | Faster | Fastest |
| **Code Review** | Difficult | Difficult | Easy |
| **Version Control** | Poor | Medium | Excellent |

---

## Auto Layout & Constraints

Auto Layout dynamically calculates the size and position of views based on constraints.

### NSLayoutConstraint

```swift
class ConstraintExampleViewController: UIViewController {
    
    private let redBox = UIView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        redBox.backgroundColor = .red
        redBox.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(redBox)
        
        // Method 1: NSLayoutConstraint individual constraints
        NSLayoutConstraint(
            item: redBox,
            attribute: .centerX,
            relatedBy: .equal,
            toItem: view,
            attribute: .centerX,
            multiplier: 1.0,
            constant: 0
        ).isActive = true
        
        NSLayoutConstraint(
            item: redBox,
            attribute: .centerY,
            relatedBy: .equal,
            toItem: view,
            attribute: .centerY,
            multiplier: 1.0,
            constant: 0
        ).isActive = true
        
        NSLayoutConstraint(
            item: redBox,
            attribute: .width,
            relatedBy: .equal,
            toItem: nil,
            attribute: .notAnAttribute,
            multiplier: 1.0,
            constant: 100
        ).isActive = true
        
        NSLayoutConstraint(
            item: redBox,
            attribute: .height,
            relatedBy: .equal,
            toItem: nil,
            attribute: .notAnAttribute,
            multiplier: 1.0,
            constant: 100
        ).isActive = true
    }
}
```

### Layout Anchors (Modern Approach)

```swift
class ModernConstraintsViewController: UIViewController {
    
    private let containerView = UIView()
    private let imageView = UIImageView()
    private let titleLabel = UILabel()
    private let descriptionLabel = UILabel()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        setupViews()
        setupConstraints()
    }
    
    private func setupViews() {
        containerView.backgroundColor = .systemGray6
        containerView.layer.cornerRadius = 12
        containerView.translatesAutoresizingMaskIntoConstraints = false
        
        imageView.contentMode = .scaleAspectFill
        imageView.clipsToBounds = true
        imageView.layer.cornerRadius = 8
        imageView.backgroundColor = .systemBlue
        imageView.translatesAutoresizingMaskIntoConstraints = false
        
        titleLabel.text = "Title"
        titleLabel.font = .systemFont(ofSize: 20, weight: .bold)
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        
        descriptionLabel.text = "Description text goes here"
        descriptionLabel.font = .systemFont(ofSize: 14)
        descriptionLabel.textColor = .gray
        descriptionLabel.numberOfLines = 0
        descriptionLabel.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(containerView)
        containerView.addSubview(imageView)
        containerView.addSubview(titleLabel)
        containerView.addSubview(descriptionLabel)
    }
    
    private func setupConstraints() {
        NSLayoutConstraint.activate([
            // Container View
            containerView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            containerView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            containerView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
            
            // Image View
            imageView.topAnchor.constraint(equalTo: containerView.topAnchor, constant: 16),
            imageView.leadingAnchor.constraint(equalTo: containerView.leadingAnchor, constant: 16),
            imageView.widthAnchor.constraint(equalToConstant: 80),
            imageView.heightAnchor.constraint(equalToConstant: 80),
            
            // Title Label
            titleLabel.topAnchor.constraint(equalTo: containerView.topAnchor, constant: 16),
            titleLabel.leadingAnchor.constraint(equalTo: imageView.trailingAnchor, constant: 12),
            titleLabel.trailingAnchor.constraint(equalTo: containerView.trailingAnchor, constant: -16),
            
            // Description Label
            descriptionLabel.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 4),
            descriptionLabel.leadingAnchor.constraint(equalTo: titleLabel.leadingAnchor),
            descriptionLabel.trailingAnchor.constraint(equalTo: titleLabel.trailingAnchor),
            descriptionLabel.bottomAnchor.constraint(lessThanOrEqualTo: imageView.bottomAnchor),
            
            // Container bottom
            containerView.bottomAnchor.constraint(equalTo: imageView.bottomAnchor, constant: 16)
        ])
    }
}
```

### Priority & Hugging/Compression

```swift
class PriorityExampleViewController: UIViewController {
    
    private let shortLabel = UILabel()
    private let longLabel = UILabel()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        setupLabels()
        setupConstraintsWithPriority()
    }
    
    private func setupLabels() {
        shortLabel.text = "Short"
        shortLabel.backgroundColor = .systemBlue
        shortLabel.translatesAutoresizingMaskIntoConstraints = false
        
        longLabel.text = "This is a very long label that should take more space"
        longLabel.backgroundColor = .systemGreen
        longLabel.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(shortLabel)
        view.addSubview(longLabel)
    }
    
    private func setupConstraintsWithPriority() {
        // Content Hugging: Resistance to stretching
        // Higher priority = more resistance to stretching
        shortLabel.setContentHuggingPriority(.required, for: .horizontal)
        longLabel.setContentHuggingPriority(.defaultLow, for: .horizontal)
        
        // Content Compression Resistance: Resistance to shrinking
        // Higher priority = more resistance to shrinking
        shortLabel.setContentCompressionResistancePriority(.defaultLow, for: .horizontal)
        longLabel.setContentCompressionResistancePriority(.required, for: .horizontal)
        
        NSLayoutConstraint.activate([
            shortLabel.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            shortLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            
            longLabel.topAnchor.constraint(equalTo: shortLabel.topAnchor),
            longLabel.leadingAnchor.constraint(equalTo: shortLabel.trailingAnchor, constant: 8),
            longLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
        ])
    }
}
```

### Stack Views

```swift
class StackViewExampleViewController: UIViewController {
    
    private let verticalStackView: UIStackView = {
        let stack = UIStackView()
        stack.axis = .vertical
        stack.spacing = 16
        stack.alignment = .fill
        stack.distribution = .fill
        stack.translatesAutoresizingMaskIntoConstraints = false
        return stack
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        setupStackView()
    }
    
    private func setupStackView() {
        view.addSubview(verticalStackView)
        
        NSLayoutConstraint.activate([
            verticalStackView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            verticalStackView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            verticalStackView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
        ])
        
        // Add arranged subviews
        let titleLabel = createLabel(text: "Title", font: .boldSystemFont(ofSize: 24))
        let subtitleLabel = createLabel(text: "Subtitle", font: .systemFont(ofSize: 16))
        
        let horizontalStack = createHorizontalButtonStack()
        
        verticalStackView.addArrangedSubview(titleLabel)
        verticalStackView.addArrangedSubview(subtitleLabel)
        verticalStackView.addArrangedSubview(horizontalStack)
        
        // Add spacing between specific views
        verticalStackView.setCustomSpacing(24, after: subtitleLabel)
    }
    
    private func createLabel(text: String, font: UIFont) -> UILabel {
        let label = UILabel()
        label.text = text
        label.font = font
        return label
    }
    
    private func createHorizontalButtonStack() -> UIStackView {
        let stack = UIStackView()
        stack.axis = .horizontal
        stack.spacing = 12
        stack.distribution = .fillEqually
        
        let cancelButton = createButton(title: "Cancel", backgroundColor: .systemGray)
        let submitButton = createButton(title: "Submit", backgroundColor: .systemBlue)
        
        stack.addArrangedSubview(cancelButton)
        stack.addArrangedSubview(submitButton)
        
        return stack
    }
    
    private func createButton(title: String, backgroundColor: UIColor) -> UIButton {
        let button = UIButton(type: .system)
        button.setTitle(title, for: .normal)
        button.setTitleColor(.white, for: .normal)
        button.backgroundColor = backgroundColor
        button.layer.cornerRadius = 8
        button.heightAnchor.constraint(equalToConstant: 50).isActive = true
        return button
    }
}
```

### Adaptive Layouts

```swift
class AdaptiveLayoutViewController: UIViewController {
    
    private let stackView = UIStackView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        setupStackView()
        updateLayoutForTraitCollection()
    }
    
    private func setupStackView() {
        stackView.spacing = 16
        stackView.distribution = .fillEqually
        stackView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stackView)
        
        NSLayoutConstraint.activate([
            stackView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            stackView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            stackView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
        ])
        
        for i in 1...3 {
            let view = UIView()
            view.backgroundColor = .systemBlue
            view.heightAnchor.constraint(equalToConstant: 100).isActive = true
            stackView.addArrangedSubview(view)
        }
    }
    
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)
        updateLayoutForTraitCollection()
    }
    
    private func updateLayoutForTraitCollection() {
        // Change stack axis based on size class
        if traitCollection.horizontalSizeClass == .compact {
            // iPhone portrait
            stackView.axis = .vertical
        } else {
            // iPhone landscape or iPad
            stackView.axis = .horizontal
        }
    }
}
```

---

## UIKit Navigation

### Navigation Controller

```swift
class HomeViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        title = "Home"
        
        setupNavigationBar()
        setupUI()
    }
    
    private func setupNavigationBar() {
        // Navigation bar appearance
        navigationController?.navigationBar.prefersLargeTitles = true
        
        // Right bar button
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            barButtonSystemItem: .add,
            target: self,
            action: #selector(addTapped)
        )
        
        // Left bar button
        navigationItem.leftBarButtonItem = UIBarButtonItem(
            image: UIImage(systemName: "gear"),
            style: .plain,
            target: self,
            action: #selector(settingsTapped)
        )
        
        // Multiple buttons
        let searchButton = UIBarButtonItem(
            image: UIImage(systemName: "magnifyingglass"),
            style: .plain,
            target: self,
            action: #selector(searchTapped)
        )
        
        let filterButton = UIBarButtonItem(
            image: UIImage(systemName: "line.3.horizontal.decrease"),
            style: .plain,
            target: self,
            action: #selector(filterTapped)
        )
        
        navigationItem.rightBarButtonItems = [filterButton, searchButton]
    }
    
    private func setupUI() {
        let pushButton = UIButton(type: .system)
        pushButton.setTitle("Push Detail", for: .normal)
        pushButton.addTarget(self, action: #selector(pushDetail), for: .touchUpInside)
        pushButton.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(pushButton)
        
        NSLayoutConstraint.activate([
            pushButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            pushButton.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
    
    @objc private func pushDetail() {
        let detailVC = DetailViewController()
        detailVC.title = "Detail"
        navigationController?.pushViewController(detailVC, animated: true)
    }
    
    @objc private func addTapped() {
        print("Add tapped")
    }
    
    @objc private func settingsTapped() {
        print("Settings tapped")
    }
    
    @objc private func searchTapped() {
        print("Search tapped")
    }
    
    @objc private func filterTapped() {
        print("Filter tapped")
    }
}

class DetailViewController: UIViewController {
    
    var data: String?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        // Hide large title for this screen
        navigationItem.largeTitleDisplayMode = .never
        
        // Custom back button
        navigationItem.leftBarButtonItem = UIBarButtonItem(
            title: "Back",
            style: .plain,
            target: self,
            action: #selector(backTapped)
        )
        
        // Or hide back button
        // navigationItem.hidesBackButton = true
    }
    
    @objc private func backTapped() {
        navigationController?.popViewController(animated: true)
    }
    
    // Intercept back button
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        
        if isMovingFromParent {
            print("Popping back")
            // Cleanup or save data
        }
    }
}
```

### Tab Bar Controller

```swift
class MainTabBarController: UITabBarController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupViewControllers()
        customizeAppearance()
    }
    
    private func setupViewControllers() {
        let homeVC = UINavigationController(rootViewController: HomeViewController())
        homeVC.tabBarItem = UITabBarItem(
            title: "Home",
            image: UIImage(systemName: "house"),
            selectedImage: UIImage(systemName: "house.fill")
        )
        
        let searchVC = UINavigationController(rootViewController: SearchViewController())
        searchVC.tabBarItem = UITabBarItem(
            title: "Search",
            image: UIImage(systemName: "magnifyingglass"),
            tag: 1
        )
        
        let profileVC = UINavigationController(rootViewController: ProfileViewController())
        profileVC.tabBarItem = UITabBarItem(
            title: "Profile",
            image: UIImage(systemName: "person"),
            selectedImage: UIImage(systemName: "person.fill")
        )
        
        // Badge
        profileVC.tabBarItem.badgeValue = "3"
        
        viewControllers = [homeVC, searchVC, profileVC]
        
        // Set default selected
        selectedIndex = 0
        
        // Delegate
        delegate = self
    }
    
    private func customizeAppearance() {
        tabBar.tintColor = .systemBlue
        tabBar.unselectedItemTintColor = .gray
        tabBar.backgroundColor = .white
        
        // iOS 15+ appearance
        let appearance = UITabBarAppearance()
        appearance.configureWithOpaqueBackground()
        appearance.backgroundColor = .white
        
        tabBar.standardAppearance = appearance
        tabBar.scrollEdgeAppearance = appearance
    }
}

extension MainTabBarController: UITabBarControllerDelegate {
    func tabBarController(_ tabBarController: UITabBarController, didSelect viewController: UIViewController) {
        print("Selected tab: \(viewController)")
    }
    
    func tabBarController(_ tabBarController: UITabBarController, shouldSelect viewController: UIViewController) -> Bool {
        // Return false to prevent selection
        return true
    }
}

class SearchViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        title = "Search"
    }
}

class ProfileViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        title = "Profile"
    }
}
```

### Modal Presentation

```swift
class ModalExampleViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        setupButton()
    }
    
    private func setupButton() {
        let button = UIButton(type: .system)
        button.setTitle("Show Modal", for: .normal)
        button.addTarget(self, action: #selector(showModal), for: .touchUpInside)
        button.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(button)
        
        NSLayoutConstraint.activate([
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            button.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
    
    @objc private func showModal() {
        let modalVC = ModalViewController()
        modalVC.delegate = self
        
        // Different presentation styles
        modalVC.modalPresentationStyle = .pageSheet // .fullScreen, .formSheet, .overFullScreen
        modalVC.modalTransitionStyle = .coverVertical // .flipHorizontal, .crossDissolve
        
        // iOS 13+ sheet customization
        if let sheet = modalVC.sheetPresentationController {
            sheet.detents = [.medium(), .large()]
            sheet.prefersGrabberVisible = true
            sheet.preferredCornerRadius = 20
        }
        
        present(modalVC, animated: true)
    }
}

protocol ModalViewControllerDelegate: AnyObject {
    func modalDidFinish(with result: String)
}

class ModalViewController: UIViewController {
    
    weak var delegate: ModalViewControllerDelegate?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        setupUI()
    }
    
    private func setupUI() {
        let closeButton = UIButton(type: .system)
        closeButton.setTitle("Close", for: .normal)
        closeButton.addTarget(self, action: #selector(closeTapped), for: .touchUpInside)
        closeButton.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(closeButton)
        
        NSLayoutConstraint.activate([
            closeButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            closeButton.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
    
    @objc private func closeTapped() {
        delegate?.modalDidFinish(with: "Data from modal")
        dismiss(animated: true)
    }
}

extension ModalExampleViewController: ModalViewControllerDelegate {
    func modalDidFinish(with result: String) {
        print("Received: \(result)")
    }
}
```

---

## TableView & CollectionView

### UITableView

```swift
class TableViewController: UIViewController {
    
    private let tableView = UITableView()
    private var items = ["Apple", "Banana", "Cherry", "Date", "Elderberry"]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupTableView()
    }
    
    private func setupTableView() {
        tableView.frame = view.bounds
        tableView.dataSource = self
        tableView.delegate = self
        
        // Register cells
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        tableView.register(CustomCell.self, forCellReuseIdentifier: "customCell")
        
        view.addSubview(tableView)
    }
}

extension TableViewController: UITableViewDataSource {
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return items.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = items[indexPath.row]
        cell.accessoryType = .disclosureIndicator
        return cell
    }
    
    // Section headers
    func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
        return "Fruits"
    }
    
    // Delete
    func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            items.remove(at: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .fade)
        }
    }
    
    // Reorder
    func tableView(_ tableView: UITableView, canMoveRowAt indexPath: IndexPath) -> Bool {
        return true
    }
    
    func tableView(_ tableView: UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath) {
        let item = items.remove(at: sourceIndexPath.row)
        items.insert(item, at: destinationIndexPath.row)
    }
}

extension TableViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        print("Selected: \(items[indexPath.row])")
    }
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 60
    }
    
    // Swipe actions
    func tableView(_ tableView: UITableView, trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
        let deleteAction = UIContextualAction(style: .destructive, title: "Delete") { [weak self] _, _, completion in
            self?.items.remove(at: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .fade)
            completion(true)
        }
        
        let editAction = UIContextualAction(style: .normal, title: "Edit") { _, _, completion in
            print("Edit")
            completion(true)
        }
        editAction.backgroundColor = .systemBlue
        
        return UISwipeActionsConfiguration(actions: [deleteAction, editAction])
    }
}

// Custom Cell
class CustomCell: UITableViewCell {
    private let titleLabel = UILabel()
    private let iconImageView = UIImageView()
    
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupUI()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setupUI() {
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        iconImageView.translatesAutoresizingMaskIntoConstraints = false
        
        contentView.addSubview(iconImageView)
        contentView.addSubview(titleLabel)
        
        NSLayoutConstraint.activate([
            iconImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            iconImageView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            iconImageView.widthAnchor.constraint(equalToConstant: 40),
            iconImageView.heightAnchor.constraint(equalToConstant: 40),
            
            titleLabel.leadingAnchor.constraint(equalTo: iconImageView.trailingAnchor, constant: 12),
            titleLabel.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            titleLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16)
        ])
    }
    
    func configure(title: String, icon: UIImage?) {
        titleLabel.text = title
        iconImageView.image = icon
    }
}
```

### Diffable Data Source (iOS 13+)

```swift
class DiffableTableViewController: UIViewController {
    
    enum Section {
        case main
    }
    
    struct Item: Hashable {
        let id: UUID
        let title: String
    }
    
    private let tableView = UITableView()
    private var dataSource: UITableViewDiffableDataSource<Section, Item>!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupTableView()
        configureDataSource()
        applyInitialSnapshots()
    }
    
    private func setupTableView() {
        tableView.frame = view.bounds
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        view.addSubview(tableView)
    }
    
    private func configureDataSource() {
        dataSource = UITableViewDiffableDataSource<Section, Item>(tableView: tableView) { tableView, indexPath, item in
            let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
            cell.textLabel?.text = item.title
            return cell
        }
    }
    
    private func applyInitialSnapshots() {
        var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
        snapshot.appendSections([.main])
        
        let items = [
            Item(id: UUID(), title: "Item 1"),
            Item(id: UUID(), title: "Item 2"),
            Item(id: UUID(), title: "Item 3")
        ]
        
        snapshot.appendItems(items)
        dataSource.apply(snapshot, animatingDifferences: true)
    }
    
    func addItem() {
        var snapshot = dataSource.snapshot()
        let newItem = Item(id: UUID(), title: "New Item")
        snapshot.appendItems([newItem])
        dataSource.apply(snapshot, animatingDifferences: true)
    }
}
```

### UICollectionView

```swift
class CollectionViewController: UIViewController {
    
    private var collectionView: UICollectionView!
    private let items = Array(1...50)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupCollectionView()
    }
    
    private func setupCollectionView() {
        let layout = createLayout()
        collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
        collectionView.dataSource = self
        collectionView.delegate = self
        collectionView.backgroundColor = .white
        collectionView.register(CustomCollectionViewCell.self, forCellWithReuseIdentifier: "cell")
        view.addSubview(collectionView)
    }
    
    private func createLayout() -> UICollectionViewLayout {
        let itemSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(0.5),
            heightDimension: .fractionalHeight(1.0)
        )
        let item = NSCollectionLayoutItem(layoutSize: itemSize)
        item.contentInsets = NSDirectionalEdgeInsets(top: 5, leading: 5, bottom: 5, trailing: 5)
        
        let groupSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0),
            heightDimension: .fractionalWidth(0.5)
        )
        let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
        
        let section = NSCollectionLayoutSection(group: group)
        section.contentInsets = NSDirectionalEdgeInsets(top: 10, leading: 10, bottom: 10, trailing: 10)
        
        let layout = UICollectionViewCompositionalLayout(section: section)
        return layout
    }
}

extension CollectionViewController: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return items.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath) as! CustomCollectionViewCell
        cell.configure(with: "\(items[indexPath.item])")
        return cell
    }
}

extension CollectionViewController: UICollectionViewDelegate {
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        print("Selected item \(items[indexPath.item])")
    }
}

class CustomCollectionViewCell: UICollectionViewCell {
    private let label = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setupUI() {
        backgroundColor = .systemBlue
        layer.cornerRadius = 8
        
        label.textAlignment = .center
        label.textColor = .white
        label.font = .boldSystemFont(ofSize: 20)
        label.translatesAutoresizingMaskIntoConstraints = false
        contentView.addSubview(label)
        
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
    }
    
    func configure(with text: String) {
        label.text = text
    }
}
```

---

## Gesture Recognizers & Touch Events

### Gesture Recognizers

```swift
class GestureViewController: UIViewController {
    
    private let box = UIView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        setupBox()
        setupGestures()
    }
    
    private func setupBox() {
        box.frame = CGRect(x: 100, y: 100, width: 100, height: 100)
        box.backgroundColor = .systemBlue
        box.layer.cornerRadius = 8
        view.addSubview(box)
    }
    
    private func setupGestures() {
        // Tap
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap))
        tapGesture.numberOfTapsRequired = 1
        box.addGestureRecognizer(tapGesture)
        
        // Double tap
        let doubleTapGesture = UITapGestureRecognizer(target: self, action: #selector(handleDoubleTap))
        doubleTapGesture.numberOfTapsRequired = 2
        box.addGestureRecognizer(doubleTapGesture)
        
        // Require double tap to fail before single tap
        tapGesture.require(toFail: doubleTapGesture)
        
        // Long press
        let longPressGesture = UILongPressGestureRecognizer(target: self, action: #selector(handleLongPress))
        longPressGesture.minimumPressDuration = 0.5
        box.addGestureRecognizer(longPressGesture)
        
        // Pan
        let panGesture = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        box.addGestureRecognizer(panGesture)
        
        // Pinch
        let pinchGesture = UIPinchGestureRecognizer(target: self, action: #selector(handlePinch))
        box.addGestureRecognizer(pinchGesture)
        
        // Rotation
        let rotationGesture = UIRotationGestureRecognizer(target: self, action: #selector(handleRotation))
        box.addGestureRecognizer(rotationGesture)
        
        // Swipe
        let swipeGesture = UISwipeGestureRecognizer(target: self, action: #selector(handleSwipe))
        swipeGesture.direction = .right
        box.addGestureRecognizer(swipeGesture)
        
        box.isUserInteractionEnabled = true
    }
    
    @objc private func handleTap(_ gesture: UITapGestureRecognizer) {
        print("Tapped")
        UIView.animate(withDuration: 0.2) {
            self.box.alpha = 0.5
        } completion: { _ in
            UIView.animate(withDuration: 0.2) {
                self.box.alpha = 1.0
            }
        }
    }
    
    @objc private func handleDoubleTap(_ gesture: UITapGestureRecognizer) {
        print("Double tapped")
        UIView.animate(withDuration: 0.3) {
            self.box.transform = self.box.transform.scaledBy(x: 1.2, y: 1.2)
        } completion: { _ in
            UIView.animate(withDuration: 0.3) {
                self.box.transform = .identity
            }
        }
    }
    
    @objc private func handleLongPress(_ gesture: UILongPressGestureRecognizer) {
        if gesture.state == .began {
            print("Long press began")
            UIView.animate(withDuration: 0.2) {
                self.box.backgroundColor = .systemRed
            }
        } else if gesture.state == .ended {
            print("Long press ended")
            UIView.animate(withDuration: 0.2) {
                self.box.backgroundColor = .systemBlue
            }
        }
    }
    
    @objc private func handlePan(_ gesture: UIPanGestureRecognizer) {
        let translation = gesture.translation(in: view)
        
        if let gestureView = gesture.view {
            gestureView.center = CGPoint(
                x: gestureView.center.x + translation.x,
                y: gestureView.center.y + translation.y
            )
        }
        
        gesture.setTranslation(.zero, in: view)
    }
    
    @objc private func handlePinch(_ gesture: UIPinchGestureRecognizer) {
        if let gestureView = gesture.view {
            gestureView.transform = gestureView.transform.scaledBy(
                x: gesture.scale,
                y: gesture.scale
            )
            gesture.scale = 1.0
        }
    }
    
    @objc private func handleRotation(_ gesture: UIRotationGestureRecognizer) {
        if let gestureView = gesture.view {
            gestureView.transform = gestureView.transform.rotated(by: gesture.rotation)
            gesture.rotation = 0
        }
    }
    
    @objc private func handleSwipe(_ gesture: UISwipeGestureRecognizer) {
        print("Swiped")
        UIView.animate(withDuration: 0.3) {
            self.box.center.x += 100
        }
    }
}
```

### Touch Events

```swift
class TouchViewController: UIViewController {
    
    private var touchPoint: CGPoint?
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesBegan(touches, with: event)
        
        guard let touch = touches.first else { return }
        touchPoint = touch.location(in: view)
        print("Touch began at \(touchPoint!)")
    }
    
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesMoved(touches, with: event)
        
        guard let touch = touches.first else { return }
        touchPoint = touch.location(in: view)
        print("Touch moved to \(touchPoint!)")
        view.setNeedsDisplay()
    }
    
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesEnded(touches, with: event)
        print("Touch ended")
        touchPoint = nil
        view.setNeedsDisplay()
    }
    
    override func touchesCancelled(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesCancelled(touches, with: event)
        print("Touch cancelled")
        touchPoint = nil
    }
}
```

---

## Accessibility in iOS Apps

### VoiceOver Support

```swift
class AccessibleViewController: UIViewController {
    
    private let imageView = UIImageView()
    private let titleLabel = UILabel()
    private let playButton = UIButton()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        setupUI()
        setupAccessibility()
    }
    
    private func setupUI() {
        imageView.image = UIImage(systemName: "photo")
        imageView.contentMode = .scaleAspectFit
        imageView.translatesAutoresizingMaskIntoConstraints = false
        
        titleLabel.text = "My Photo Album"
        titleLabel.font = .boldSystemFont(ofSize: 24)
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        
        playButton.setTitle("Play", for: .normal)
        playButton.setTitleColor(.white, for: .normal)
        playButton.backgroundColor = .systemBlue
        playButton.layer.cornerRadius = 8
        playButton.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(imageView)
        view.addSubview(titleLabel)
        view.addSubview(playButton)
        
        NSLayoutConstraint.activate([
            imageView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            imageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            imageView.widthAnchor.constraint(equalToConstant: 200),
            imageView.heightAnchor.constraint(equalToConstant: 200),
            
            titleLabel.topAnchor.constraint(equalTo: imageView.bottomAnchor, constant: 20),
            titleLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            
            playButton.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 20),
            playButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            playButton.widthAnchor.constraint(equalToConstant: 120),
            playButton.heightAnchor.constraint(equalToConstant: 50)
        ])
    }
    
    private func setupAccessibility() {
        // Enable accessibility
        imageView.isAccessibilityElement = true
        titleLabel.isAccessibilityElement = true
        playButton.isAccessibilityElement = true
        
        // Labels
        imageView.accessibilityLabel = "Photo album cover"
        titleLabel.accessibilityLabel = "Album title: My Photo Album"
        playButton.accessibilityLabel = "Play slideshow"
        
        // Hints
        imageView.accessibilityHint = "Tap to view full image"
        playButton.accessibilityHint = "Double tap to start playing photos"
        
        // Traits
        playButton.accessibilityTraits = [.button]
        titleLabel.accessibilityTraits = [.header]
        
        // Values (for dynamic content)
        // playButton.accessibilityValue = "Paused"
        
        // Custom actions
        let deleteAction = UIAccessibilityCustomAction(
            name: "Delete album",
            target: self,
            selector: #selector(deleteAlbum)
        )
        
        imageView.accessibilityCustomActions = [deleteAction]
        
        // Group elements
        view.shouldGroupAccessibilityChildren = true
        
        // Navigation order
        view.accessibilityElements = [titleLabel, imageView, playButton]
    }
    
    @objc private func deleteAlbum() -> Bool {
        print("Delete album")
        return true
    }
    
    // Respond to accessibility notifications
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        UIAccessibility.post(
            notification: .screenChanged,
            argument: titleLabel
        )
    }
}
```

### Dynamic Type Support

```swift
class DynamicTypeViewController: UIViewController {
    
    private let titleLabel: UILabel = {
        let label = UILabel()
        label.text = "Dynamic Type Example"
        label.font = .preferredFont(forTextStyle: .largeTitle)
        label.adjustsFontForContentSizeCategory = true
        label.numberOfLines = 0
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()
    
    private let bodyLabel: UILabel = {
        let label = UILabel()
        label.text = "This text will scale based on user's preferred text size settings."
        label.font = .preferredFont(forTextStyle: .body)
        label.adjustsFontForContentSizeCategory = true
        label.numberOfLines = 0
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        view.addSubview(titleLabel)
        view.addSubview(bodyLabel)
        
        NSLayoutConstraint.activate([
            titleLabel.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            titleLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            titleLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            
            bodyLabel.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 20),
            bodyLabel.leadingAnchor.constraint(equalTo: titleLabel.leadingAnchor),
            bodyLabel.trailingAnchor.constraint(equalTo: titleLabel.trailingAnchor)
        ])
    }
}
```

### Interview Questions

**Q1: What's the difference between TableView and CollectionView?**

**Answer:** TableView is for simple vertical lists with one column. CollectionView is more flexible, supporting grid layouts, horizontal scrolling, and custom layouts. Use TableView for simple lists, CollectionView for complex layouts.

**Q2: When should you use programmatic UI vs Storyboards?**

**Answer:** Use programmatic UI for better version control, reusability, and team collaboration. Use Storyboards for prototyping, simple apps, or when working solo. Larger teams typically prefer programmatic UI.

**Q3: How do you make your app accessible?**

**Answer:** 
- Add accessibility labels, hints, and traits
- Support Dynamic Type
- Ensure sufficient color contrast
- Test with VoiceOver
- Support keyboard navigation
- Provide alternative text for images

---

[← Previous: OOP and POP](oop-and-pop.md) | [Next: SwiftUI →](swiftui.md)

[Back to Main](../README.md)


## Interview Questions & Answers

### Q1: What's the difference between TableView and CollectionView?

**Answer:**

**UITableView:**
- Single column, vertical scrolling
- Built-in cell styles
- Simpler to implement
- Best for lists
- Row-based layout
- Optimized for vertical lists

**UICollectionView:**
- Multi-column, any direction
- Custom layouts required
- More flexible
- Best for grids, galleries
- Item-based layout
- Supports complex layouts (waterfall, circular)

**When to Choose:**
- Use TableView: Settings, contacts, messages
- Use CollectionView: Photo galleries, product catalogs, calendars

### Q2: Explain Auto Layout constraints priority

**Answer:**

**Priority Levels:**
- `required` (1000) - Must be satisfied
- `defaultHigh` (750) - Should be satisfied
- `defaultLow` (250) - Can be broken
- Custom (1-999)

**Example:**

```swift
// View should be 100 wide, but can shrink if needed
widthConstraint.priority = .defaultHigh  // 750
widthConstraint.constant = 100

// Minimum width is required
minWidthConstraint.priority = .required  // 1000
minWidthConstraint.constant = 50
```

**Content Hugging** (resist stretching):
- Higher = more resistance to growing

**Content Compression Resistance** (resist shrinking):
- Higher = more resistance to shrinking

```swift
label1.setContentHuggingPriority(.required, for: .horizontal)
label2.setContentCompressionResistancePriority(.defaultLow, for: .horizontal)
```

### Q3: How do you optimize TableView performance?

**Answer:**

**1. Cell Reuse:**
```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
    return cell
}
```

**2. Cache Heights:**
```swift
var heightCache: [IndexPath: CGFloat] = [:]

func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
    if let cached = heightCache[indexPath] {
        return cached
    }
    let height = calculateHeight(for: indexPath)
    heightCache[indexPath] = height
    return height
}
```

**3. Estimate Heights:**
```swift
tableView.estimatedRowHeight = 100
tableView.rowHeight = UITableView.automaticDimension
```

**4. Avoid Heavy Operations in cellForRow:**
- Load images asynchronously
- Cache processed data
- Minimize subview hierarchy

**5. Opaque Views:**
```swift
cell.contentView.isOpaque = true
cell.backgroundColor = .white
```

### Q4: What are the different types of segues?

**Answer:**

**1. Show (Push):**
- Pushes onto navigation stack
- Back button automatically added
- Used in UINavigationController

**2. Show Detail:**
- Replaces detail view in UISplitViewController
- On iPhone, acts like Show

**3. Present Modally:**
- Presents over current view
- Various presentation styles
- Must dismiss explicitly

**4. Present as Popover:**
- Shows in popover on iPad
- Modal on iPhone

**5. Custom:**
- Custom transition animations

**Programmatic Alternative:**
```swift
// Push
navigationController?.pushViewController(vc, animated: true)

// Modal
present(vc, animated: true)

// Popover
vc.modalPresentationStyle = .popover
present(vc, animated: true)
```

### Q5: How do you handle memory warnings in iOS?

**Answer:**

**1. Implement didReceiveMemoryWarning:**
```swift
override func didReceiveMemoryWarning() {
    super.didReceiveMemoryWarning()
    
    // Clear caches
    imageCache.removeAll()
    dataCache.removeAll()
    
    // Release recreatable resources
    heavyObjects = nil
}
```

**2. Implement in AppDelegate:**
```swift
func applicationDidReceiveMemoryWarning(_ application: UIApplication) {
    // Clear app-wide caches
    URLCache.shared.removeAllCachedResponses()
    
    // Notify view controllers
    NotificationCenter.default.post(name: .memoryWarning, object: nil)
}
```

**3. Monitor Memory:**
- Use Instruments (Allocations tool)
- Watch for memory growth
- Test on real devices

**4. Prevention:**
- Use autoreleasepool for loops
- Release large objects quickly
- Implement proper image caching
- Lazy load resources

### Q6: Explain the responder chain in iOS

**Answer:**

The responder chain is how iOS handles events (touches, shakes, etc.).

**Chain Order:**
```
UIView → UIViewController → UIWindow → UIApplication → AppDelegate
```

**How it Works:**
1. Event hits a view
2. If view doesn't handle it, passes to next responder
3. Continues until handled or reaches AppDelegate

**Example:**

```swift
class CustomView: UIView {
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        if shouldHandleTouch {
            // Handle touch
        } else {
            // Pass to next responder
            super.touchesBegan(touches, with: event)
        }
    }
    
    override var next: UIResponder? {
        return superview
    }
}
```

**Practical Use:**
- Gesture handling
- Keyboard events
- Motion events
- Menu actions

### Q7: What's the difference between frame and layoutSubviews?

**Answer:**

**frame:**
- Property that defines position and size
- Relative to superview
- Setting frame triggers layout

```swift
view.frame = CGRect(x: 0, y: 0, width: 100, height: 100)
```

**layoutSubviews():**
- Method called when layout is needed
- Override to perform custom layout
- Called automatically, never call directly
- Use setNeedsLayout() to trigger

```swift
class CustomView: UIView {
    override func layoutSubviews() {
        super.layoutSubviews()
        
        // Perform custom layout
        subview1.frame = CGRect(x: 0, y: 0, width: bounds.width/2, height: bounds.height)
        subview2.frame = CGRect(x: bounds.width/2, y: 0, width: bounds.width/2, height: bounds.height)
    }
}

// Trigger layout
view.setNeedsLayout()  // Mark as needing layout
view.layoutIfNeeded()  // Force layout immediately
```

**When layoutSubviews is Called:**
- View size changes
- Subview added/removed
- Scroll view scrolls
- Device rotates
- Constraints change

---

[← Previous: OOP and POP](oop-and-pop.md) | [Next: SwiftUI →](swiftui.md)

[Back to Main](../README.md)
