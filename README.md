## 1\. To Do List  앱의 기능

To Do List 앱은 할 일 목록을 관리하는 앱으로 UITableView를 이용하여 할 일 목록을 표시하고, 사용자는 UIBarButtonItems를 이용하여 추가, 수정, 삭제 등의 작업을 수행할 수 있습니다.

또한 UserDefaults를 이용하여 할 일 목록을 저장하고 로드하므로, 앱을 종료한 후 다시 실행하더라도 저장된 할 일 목록을 유지할 수 있습니다. 

사용자는 Add 버튼을 눌러 할 일을 등록할 수 있으며, 등록한 할 일은 UITableView에 나타나고, 사용자는 할 일을 선택하여 완료 여부를 표시할 수 있습니다.

사용자는 편집 버튼을 눌러 편집 모드로 전환하여 할 일 목록을 수정하거나  삭제할 수 있습니다. 이때, done 버튼을 누르면 편집 모드가 해제되고, 수정된 할 일 목록이 저장됩니다.

## 2\. 코드 분석

### 2-1. View Controller 클래스

ViewController 클래스는 UIViewController를 상속받으며, UITableViewDelegate, UITableViewDataSource 프로토콜을 채택합니다. 

```
ViewController.swift

class ViewController: UIViewController {
	// ...
}
```

### 2-2. 할 일(Task) 구조체

할 일 목록의 각 항목을 관리하기 위해, Task 라는 이름의 구조체를 정의합니다. Task 구조체는 할 일의 제목과 완료 여부(done)를 저장합니다.

```
Task.swift

import Foudantion

struct Task {
    var title: String
    var done: Bool
}
```

### 2-3. IBOutlet 및 변수 정의

View Controller 클래스 안에는 IBOutlet으로 연결된 UI 요소와 함께, 할 일 목록을 저장하기 위한 tasks 배열과 doneButton 변수가 정의 됩니다.

[##_Image|kage@XcCyO/btr2ugMHXQa/NtVHyRGkTkCDiDNk08kAwK/img.png|CDM|1.3|{"originWidth":1838,"originHeight":1110,"style":"alignCenter","caption":"인터페이스 객체 코드 연결"}_##]

```
class ViewController: UIViewController {
    @IBOutlet var editButton: UIBarButtonItem!
    @IBOutlet weak var tableView: UITableView!
    var doneButton: UIBarButtonItem?
    var tasks = [Task]() {
    	didSet {
        	self.saveTasks()
        }
    }
    // ...
}
```

\- editButton : 편집 버튼 입니다.

\- tableView : 할 일 목록을 표시하는 테이블 뷰 입니다.

\- doneButton : 편집 모드에서 done 버튼입니다.

\- tasks : 할 일 목록을 저장하는 배열입니다.

###   
2-4. viewDidLoad()

View Controller 클래스의 viewDidLoad() 함수에서는, doneButton 변수에 UIBarButtonItem 인스턴스를 생성하고, tableView의 dataSource와 [delegate](https://zeddios.tistory.com/8)를 ViewController로 설정합니다. 그리고, 이전에 저장한 할 일 목록을 불러와서 tasks 배열에 저장합니다.

```
override func viewDidLoad() {
    super.viewDidLoad()
    self.doneButton = UIBarButtonItem(barButtonSystemItem: .done, target: self, action: #selector(doneButtonTap))
    self.tableView.dataSource = self
    self.tableView.delegate = self
    self.loadTasks()
}
```

### 2-5. doneButtonTap()

완료(done) 버튼을 눌렀을 때 호출되며, tableView의 setEditing(\_:animated:) 메서드를 이용하여 편집 모드를 해제합니다.

```
@objc func doneButtonTap() {
    self.navigationItem.leftBarButtonItem = self.editButton
    self.tableView.setEditing(false, animated: true)
}
```

### 2-6. tapEditButton()

편집 버튼을 눌렀을 때 호출되며, tableView의 setEditing(\_:animated:) 메서드를 이용하여 편집 모드를 설정합니다. 만약, tasks 배열이 비어있으면 편집모드를 설정하지 않습니다. setEditing(\_:animated:) 메서드는 편집모드에 집입하거나 해제할 때, 애니메이션을 보여줍니다. 

```
@IBAction func tapEditButton(_ sender: UIBarButtonItem) {
    guard !self.tasks.isEmpty else { return }
    self.navigationItem.leftBarButtonItem = self.doneButton
    self.tableView.setEditing(true, animated: true)
}
```

### 2-7. tapAddButton()

할 일을 추가하기 위해, Add 버튼을 누르면 UIAlertController를 이용한 팝업창이 나타납니다. 팝업창에서 사용자가 할 일을 입력하고 등록 버튼을 누르면, tasks 배열에 새로운 할 일 객체를 추가합니다.

```
@IBAction func tapAddButton(_ sender: UIBarButtonItem) {
    let alert = UIAlertController(title: "할 일 등록", message: "할 일을 입력해주세요.", preferredStyle: .alert)
    let registerButton =  UIAlertAction(title: "등록", style: .default, handler: { [weak self] _ in
        guard let title = alert.textFields?[0].text else { return }
        let task = Task(title: title, done: false)
        self?.tasks.append(task)
        self?.tableView.reloadData()
    })
    let cancelButton = UIAlertAction(title: "취소", style: .cancel, handler: nil)
    alert.addAction(registerButton)
    alert.addAction(cancelButton)
    alert.addTextField(configurationHandler: { textFiled in
        textFiled.placeholder = "할 일을 입력해주세요."
    })
    self.present(alert, animated: true, completion: nil)
}
```

\- UIAlertController : UIAlertController를 이용하여 팝업창을 띄웁니다.

\- UIAlertAction : 등록 버튼과 취소 버튼을 추가합니다.

\- addTextField() : 팝업창 내부에 UITextField를 추가합니다.

### 2-8. saveTasks(), loadTask()

UserDefaults를 이용하여 tasks 배열을 저장하고 불러올 수 있습니다.

```
func saveTasks() {
    let data = self.tasks.map {
        [
            "title": $0.title,
            "done": $0.done
        ]
    }
    let userDefaults = UserDefaults.standard
    userDefaults.set(data, forKey: "tasks")
}

func loadTasks() {
    let userDefaults = UserDefaults.standard
    guard let data = userDefaults.object(forKey: "tasks") as? [[String: Any]] else { return }
    self.tasks = data.compactMap {
        guard let title = $0["title"] as? String else { return nil }
        guard let done = $0["done"] as? Bool else { return nil }
        return Task(title: title, done: done)
    }
}
```

\- saveTasks(): tasks 배열을 UserDefaults에 저장합니다.

\- loadTasks(): UserDefaults에서 tasks 배열을 불러옵니다.

설명 : [※ userDefault](https://jamong-ios.tistory.com/6), ※compactMap

### 2-9. UITableViewDataSource

UITableViewDataSource 프로토콜의 메서드를 구현하여, 할 일 목록을 표시하고 관리합니다.

```
extension ViewController: UITableViewDataSource {
    // 섹션 내의 행 수 반환
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.tasks.count
    }
    
    // 각 셀을 구성하고 반환
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
        let task = self.tasks[indexPath.row]
        cell.textLabel?.text = task.title
        if task.done {
            cell.accessoryType = .checkmark
        } else{
            cell.accessoryType = .none
        }
        return cell
    }
    
    // 편집 모드에서 삭제 버튼 누를 때 호출, tasks 배열에서 해당하는 할 일 삭제
    func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        self.tasks.remove(at: indexPath.row)
        tableView.deleteRows(at: [indexPath], with: .automatic)
        
        if self.tasks.isEmpty {
            self.doneButtonTap()
        }
    }
    
    // 각 셀의 이동 가능 여부를 반환
    func tableView(_ tableView: UITableView, canMoveRowAt indexPath: IndexPath) -> Bool {
        return true
    }
    
    // 셀을 이동할 때 호출, tasks 배열에서 해당하는 할 일의 위치를 변경
    func tableView(_ tableView: UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath) {
        var tasks = self.tasks
        let task = tasks[sourceIndexPath.row]
        tasks.remove(at: sourceIndexPath.row)
		tasks.insert(task, at: destinationIndexPath.row)
		self.tasks = tasks
	}
}
```

\- tableView(\_:numberOfRowsInSection:) : 섹션 내의 행 수를 반환합니다.

\- tableView(\_:cellForRowAt:) : 각 셀을 구성하고 반환합니다.

\- tableView(\_:commit:forRowAt:) : 사용자가 편집 모드에서 삭제 버튼을 누를 때 호출되며, tasks 배열에서 해당하는 할 일을 삭제합니다.

\- tableView(\_:canMoveRowAt:) : 각 셀의 이동 가능 여부를 반환합니다.

\- tableView(\_:moveRowAt:to:) : 사용자가 셀을 이동할 때 호출되며, tasks 배열에서 해당하는 할 일의 위치를 변경합니다.

### 2-10. UITableViewDelegate

UITableViewDelegate 프로토콜의 메서드를 구현하며, 할 일의 완료 여부를 표시합니다.

```
extension ViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        var task = self.tasks[indexPath.row]
        task.done = !task.done
        self.tasks[indexPath.row] = task
        self.tableView.reloadRows(at: [indexPath], with: .automatic)
    }
}
```

\- tableView(\_:didSelectRowAt:) : 셀을 선택하면 호출되면, 해당하는 할 일의 완료 여부를 토글합니다.

설명 : ※Delegate

설명 부분에 하이퍼링크가 안 달려있는 곳은 글 생성 후 연결 해주려고합니다.