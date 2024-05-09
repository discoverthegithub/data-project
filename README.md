#include "class.h"
#include<windows.h>

int HashTable::hash(const string& key) const
{
    int hashValue = 0;
    for (char c : key)
    {
        hashValue += static_cast<int>(c);
    }
    return hashValue % table_size;
}

int HashTable::gethash(string& key)
{
    return hash(key);
}
HashTable::HashTable()
{
    for (int i = 0; i < table_size; ++i)
    {
        table[i] = nullptr;
    }
}

HashTable::~HashTable()
{
    for (int i = 0; i < table_size; ++i)
    {
        Node* current = table[i];
        while (current != nullptr)
        {
            Node* temp = current;
            current = current->next;
            delete temp;
        }
    }
}

void HashTable::insert(const User& user)
{
    int index = hash(user.username);
    Node* newNode = new Node(user);
    newNode->next = table[index];
    table[index] = newNode;
}

User* HashTable::search(const string& username) const
{
    int index = hash(username);
    Node* current = table[index];
    while (current != nullptr)
    {
        if (current->user.username == username)
        {
            return &(current->user);
        }
        current = current->next;
    }
    return nullptr;
}

UserManagement::UserManagement() : user_count(0), filename("users.csv"), hashTable()
{
    loadUsers();
}

UserManagement::UserManagement(const string& file) : user_count(0), filename(file), hashTable()
{
    loadUsers();
}

void UserManagement::loadUsers()
{
    ifstream file(filename);
    if (file.is_open())
    {
        string line;
        while (getline(file, line) && user_count < max_user)
        {
            size_t commaIndex = line.find(',');
            string username = line.substr(0, commaIndex);
            string password = line.substr(commaIndex + 1);
            hashTable.insert(User(username, password));
            users[user_count].username = username;
            users[user_count].password = password;
            ++user_count;
        }
        file.close();
    }
}


void UserManagement::saveUsers() const
{
    ofstream file(filename);
    if (file.is_open())
    {
        for (int i = 0; i < user_count; ++i)
        {
            file << users[i].username << "," << users[i].password << endl;
        }
        file.close();
    }
    else
    {
        cout << "Error: Could not create or open file for writing." << endl;
        Sleep(1000);
    }
}


bool UserManagement::registerUser(const string& username, const string& password)
{
    if (user_count < max_user)
    {
        for (int i = 0; i < max_user; ++i)
        {
            if (users[i].username == username)
            {
                cout << "Username already exists. Please choose a different one." << endl;
                Sleep(1000);
                return false;
            }
        }
        for (int i = 0; i < max_user; ++i)
        {
            if (users[i].username.empty())
            {
                users[i].username = username;
                users[i].password = password;
                ++user_count;
                saveUsers();
                cout << "User " << username << " registered successfully." << endl;
                Sleep(1000);
                return true;
            }
        }
    }
    else
    {
        cout << "Maximum number of users reached. Cannot register new user." << endl;
        Sleep(1000);
    }
    return false;
}

bool UserManagement::loginUser(const string& username, const string& password)
{
    User* user = hashTable.search(username);
    if (user != nullptr && user->password == password)
    {
        user->loggedIn = true;
        saveUsers(); // Save user data after logging in
        cout << "User " << username << " logged in successfully." << endl;
        Sleep(1000);
        return true;
    }
    cout << "Invalid username or password." << endl;
    Sleep(1000);
    return false;
}

void UserManagement::logoutUser(const string& username)
{
    User* user = hashTable.search(username);
    if (user != nullptr)
    {
        user->loggedIn = false;
        saveUsers(); // Save user data after logging out
        cout << "User " << username << " logged out successfully." << endl;
        Sleep(1000);
    }
    else
    {
        cout << "User not found." << endl;
        Sleep(1000);
    }
}


User* UserManagement::getUser(const string& username)
{
    return hashTable.search(username);
}
Node* AVLTree::rotateRight(Node* y)
{
    Node* x = y->left;
    Node* T = x->right;

    x->right = y;
    y->left = T;

    return x;
}

Node* AVLTree::rotateLeft(Node* x)
{
    Node* y = x->right;
    Node* T = y->left;

    y->left = x;
    x->right = T;

    return y;
}

int AVLTree::getHeight(Node* node)
{
    if (node == nullptr)
        return 0;
    return max(getHeight(node->left), getHeight(node->right)) + 1;
}

int AVLTree::getBalanceFactor(Node* node)
{
    if (node == nullptr)
        return 0;
    return getHeight(node->left) - getHeight(node->right);
}

Node* AVLTree::insertNode(Node* node, const string& value)
{
    if (node == nullptr)
    {
        node = new Node();
        node->project = value;
        return node;
    }

    if (value < node->project)
        node->left = insertNode(node->left, value);
    else if (value > node->project)
        node->right = insertNode(node->right, value);
    else
        return node;

    int balance = getBalanceFactor(node);

    if (balance > 1 && value < node->left->project)
        return rotateRight(node);

    if (balance < -1 && value > node->right->project)
        return rotateLeft(node);

    if (balance > 1 && value > node->left->project) {
        node->left = rotateLeft(node->left);
        return rotateRight(node);
    }

    if (balance < -1 && value < node->right->project) {
        node->right = rotateRight(node->right);
        return rotateLeft(node);
    }

    return node;
}

Node* AVLTree::findMin(Node* node)
{
    if (node == nullptr)
        return nullptr;
    while (node->left != nullptr)
        node = node->left;
    return node;
}

Node* AVLTree::deleteNode(Node* node, const string& value)
{
    if (node == nullptr)
        return node;

    if (value < node->project)
        node->left = deleteNode(node->left, value);
    else if (value > node->project)
        node->right = deleteNode(node->right, value);
    else {
        if (node->left == nullptr || node->right == nullptr)
        {
            Node* temp = node->left ? node->left : node->right;
            if (temp == nullptr)
            {
                temp = node;
                node = nullptr;
            }
            else
                *node = *temp;
            delete temp;
        }
        else
        {
            Node* temp = findMin(node->right);
            node->project = temp->project;
            node->right = deleteNode(node->right, temp->project);
        }
    }

    if (node == nullptr)
        return node;

    int balance = getBalanceFactor(node);

    if (balance > 1 && getBalanceFactor(node->left) >= 0)
        return rotateRight(node);

    if (balance > 1 && getBalanceFactor(node->left) < 0) {
        node->left = rotateLeft(node->left);
        return rotateRight(node);
    }

    if (balance < -1 && getBalanceFactor(node->right) <= 0)
        return rotateLeft(node);

    if (balance < -1 && getBalanceFactor(node->right
    ) > 0) {
        node->right = rotateRight(node->right);
        return rotateLeft(node);
    }

    return node;
}

void AVLTree::inorderTraversal(Node* node)
{
    if (node != nullptr)
    {
        inorderTraversal(node->left);
        cout << node->project << endl;
        inorderTraversal(node->right);
    }
}

void AVLTree::inorderTraversalToFile(Node* node, ofstream& file)
{
    if (node != nullptr) {
        inorderTraversalToFile(node->left, file);
        file << node->project << endl;
        inorderTraversalToFile(node->right, file);
    }
}

AVLTree::AVLTree() : root(nullptr) {}

void AVLTree::addFile(const string& value)
{
    root = insertNode(root, value);
}

void AVLTree::deleteFile(const string& value) {
    root = deleteNode(root, value);
}

void AVLTree::displayRepository()
{
    inorderTraversal(root);
}

Repository::Repository() : current_index(0)
{
    for (int i = 0; i < 100; ++i)
        repositories[i] = nullptr;
}

void Repository::createRepository(const string& name, int hashVal)
{
    // Construct file names based on the hash value
    string repoFileName = "repositoryname" + to_string(hashVal) + ".txt";
    string privateRepoFileName = "privaterepository" + to_string(hashVal) + ".txt";

    // Open repository file to count the number of lines
    ifstream repoNamesFile(repoFileName);
    int lineCount = 0;
    string line;
    while (getline(repoNamesFile, line))
    {
        if (!line.empty()) {
            lineCount++;
        }
    }
    repoNamesFile.close();

    // Set currentIndex based on the line count
    int currentIndex = lineCount;

    if (currentIndex >= 100)
    {
        cout << "Cannot create more repositories. Maximum limit reached." << endl;
        Sleep(1000);
        return;
    }

    // Ask the user whether the repository should be private or public
    int privacyChoice;
    do
    {
        cout << "Do you want to make the repository private or public?" << endl;
        cout << "Enter 1 for public, 0 for private: ";
        cin >> privacyChoice;
        if (privacyChoice == 0 || privacyChoice == 1)
        {
            break; // Valid choice, exit loop
        }
        else
        {
            cout << "Invalid choice. Please enter 1 for public or 0 for private." << endl;
            Sleep(1000);
        }
    } while (true);

    string password;
    if (privacyChoice == 0)
    {
        cout << "Enter the password for the private repository: ";
        cin >> password;
        Sleep(1000);
    }

    // Store the repository name in repositorynames.txt
    ofstream repoNamesFileAppend(repoFileName, ios::app);
    if (!repoNamesFileAppend.is_open())
    {
        cout << "Error: Unable to open " << repoFileName << " for writing." << endl;
        Sleep(1000);
        return;
    }
    repoNamesFileAppend << name << endl;
    repoNamesFileAppend.close();

    // Store the repository name and password (if private) in privaterepository.txt
    if (privacyChoice == 0)
    {
        ofstream privateRepoFile(privateRepoFileName, ios::app);
        if (!privateRepoFile.is_open()) {
            cout << "Error: Unable to open " << privateRepoFileName << " for writing." << endl;
            Sleep(1000);
            return;
        }
        privateRepoFile << name << "," << password << endl;
        Sleep(1000);
        privateRepoFile.close();
    }

    // Store the repository at the current index and increment currentIndex
    repositories[currentIndex] = new AVLTree(); // Assuming this line was here in your original code
    cout << "Repository '" << name << "' created at index: " << currentIndex << endl;
    Sleep(1000);
    currentIndex++;

}

void Repository::addFileToRepository(int hashVal)
{
    // Construct repository file name based on the hash value
    string repoFileName = "repositoryname" + to_string(hashVal) + ".txt";

    // Open repository file to find the repository names
    ifstream repoNameFile(repoFileName);
    string repoName;
    string repositories[100]; // Array to store repository names
    int repoCount = 0; // Variable to keep track of the number of repositories

    if (repoNameFile.is_open())
    {
        while (getline(repoNameFile, repoName))
        {
            repositories[repoCount] = repoName; // Store repository name in the array
            repoCount++;
            if (repoCount >= 100) // Ensure not to exceed the array size
                break;
        }
        repoNameFile.close();
    }
    else
    {
        cout << "Error: Unable to open " << repoFileName << "." << endl;
        Sleep(1000);
        return;
    }

    // Display all the repositories
    cout << "Repositories:" << endl;
    for (int i = 0; i < repoCount; ++i)
    {
        cout << i << ": " << repositories[i] << endl;
    }

    // Ask the user to select a repository index
    int index;
    cout << "Enter the index of the repository to add the file: ";
    cin >> index;
    cin.ignore(); // Ignore any newline characters in the input buffer

    // Check if the selected index is valid
    if (index < 0 || index >= repoCount)
    {
        cout << "Invalid repository index." << endl;
        return;
    }

    string projectName;
    cout << "Enter the name of the project to add: ";
    getline(cin, projectName);

    // Construct repository file name with the selected repository name
    string repositoryFileName = repositories[index] + ".txt";

    // Open the repository file for the selected repository name
    ofstream repositoryFile(repositoryFileName, ios::app);
    if (!repositoryFile.is_open())
    {
        cout << "Error: Unable to open repository file for writing." << endl;
        Sleep(1000);
        return;
    }

    // Add the project name to the repository file
    repositoryFile << projectName << endl;
    repositoryFile.close();

    // Create a file with the project name and ask the user to input the content
    ofstream projectFile(projectName + ".txt");

    if (projectFile.is_open())
    {
        cout << "Enter the content of the file for project " << projectName << ":" << endl;
        string content;
        getline(cin, content); // Get the content from the user
        projectFile << content << endl; // Write the content to the file
        projectFile.close();
        cout << "File added to repository '" << repositories[index] << "'." << endl;
        Sleep(1000);
    }
    else
    {
        cout << "Error: Unable to create project file." << endl;
        Sleep(1000);
    }
}




void Repository::deleteFileFromRepository(int index, int hashVal)
{
    if (index < 0 || index >= 100) {
        cout << "Invalid repository index." << endl;
        Sleep(1000);
        return;
    }

    // Construct repository file name based on the hash value
    string repoFileName = "repositoryname" + to_string(hashVal) + ".txt";

    ifstream repoNameFile(repoFileName);
    string repoName;
    if (repoNameFile)
    {
        for (int i = 0; i <= index; ++i)
        {
            getline(repoNameFile, repoName);
        }
        repoNameFile.close();
    }
    else
    {
        cout << "Error: Unable to open " << repoFileName << "." << endl;
        Sleep(1000);
        return;
    }

    ifstream repositoryFile(repoName + ".txt");
    if (!repositoryFile)
    {
        cout << "Error: Unable to open repository file for reading." << endl;
        Sleep(1000);
        return;
    }

    cout << "Contents of repository '" << repoName << "' at index " << index << ":" << endl;
    Sleep(1000);

    string projectName;
    int projectIndex = 0;
    while (getline(repositoryFile, projectName))
    {
        cout << ++projectIndex << ". " << projectName << endl;
    }
    repositoryFile.close();

    int projectNumber;
    do
    {
        cout << "Enter the project number you want to delete (0 to return to the main menu): ";
        cin >> projectNumber;
        if (projectNumber == 0)
        {
            return; // Return to main menu
        }
        else if (projectNumber < 1 || projectNumber > projectIndex)
        {
            cout << "Invalid project number. Please enter a valid project number." << endl;
            Sleep(1000);
        }
        else
        {
            ifstream projectFile(repoName + ".txt");
            if (projectFile)
            {
                string projectLine;
                for (int i = 0; i < projectNumber; ++i)
                {
                    getline(projectFile, projectLine);
                }
                projectFile.close();

                string projectFileName = projectLine + ".txt";
                if (remove(projectFileName.c_str()) != 0)
                {
                    cerr << "Error: Unable to delete project file." << endl;
                    Sleep(1000);
                    return;
                }

                ifstream repoFileIn(repoName + ".txt");
                if (!repoFileIn) {
                    cout << "Error: Unable to open repository file for reading." << endl;
                    Sleep(1000);
                    return;
                }

                ofstream repoFileOut(repoName + "_temp.txt");
                if (!repoFileOut)
                {
                    cout << "Error: Unable to open repository file for writing." << endl;
                    Sleep(1000);
                    return;
                }

                string line;
                int currentLine = 0;
                while (getline(repoFileIn, line))
                {
                    ++currentLine;
                    if (currentLine != projectNumber)
                    {
                        repoFileOut << line << endl;
                    }
                }
                repoFileIn.close();
                repoFileOut.close();

                if (remove((repoName + ".txt").c_str()) != 0)
                {
                    cout << "Error: Unable to remove original repository file." << endl;
                    Sleep(1000);
                    return;
                }
                if (rename((repoName + "_temp.txt").c_str(), (repoName + ".txt").c_str()) != 0)
                {
                    cout << "Error: Unable to rename temp repository file." << endl;
                    Sleep(1000);
                    return;
                }

                cout << "Project deleted successfully." << endl;
                Sleep(1000);
                return;
            }
            else
            {
                cout << "Error: Unable to open project file for reading." << endl;
                Sleep(1000);
                return;
            }
        }
    } while (true);
}


string Repository::getLineFromFile(const string& fileName, int lineNum)
{
    ifstream file(fileName);
    string line;
    for (int i = 0; i < lineNum; ++i)
    {
        if (!getline(file, line))
        {
            return ""; // Return empty string if lineNum exceeds the number of lines in the file
        }
    }
    return line;
}

void Repository::displayRepository(int hashVal)
{
    // Construct repository file name based on the hash value
    string repoFileName = "repositoryname" + to_string(hashVal) + ".txt";

    // Open repository file to find the repository names
    ifstream repoNameFile(repoFileName);
    if (!repoNameFile.is_open())
    {
        cout<< "Error: Unable to open " << repoFileName << "." << endl;
        Sleep(1000);
        return;
    }

    string repoName;
    string repositories[100]; // Array to store repository names
    int repoCount = 0; // Variable to keep track of the number of repositories

    // Read repository names from file
    while (getline(repoNameFile, repoName) && repoCount < 100)
    {
        repositories[repoCount] = repoName; // Store repository name in the array
        ++repoCount;
    }
    repoNameFile.close();

    // Display all the repositories
    cout << "Repositories:" << endl;
    for (int i = 0; i < repoCount; ++i)
    {
        cout << i + 1 << ": " << repositories[i] << endl;
    }

    // Ask the user to select a repository index
    int index;
    cout << "Enter the index of the repository to view its contents (0 to return to the main menu): ";
    cin >> index;
    cin.ignore(); // Ignore any newline characters in the input buffer

    // Check if the selected index is valid
    if (index < 1 || index > repoCount)
    {
        cout << "Invalid repository index." << endl;
        return;
    }

    // Display contents of the selected repository
    string selectedRepo = repositories[index - 1];
    displayRepositoryContent(selectedRepo);
}

void Repository::displayRepositoryContent(const string& repoName)
{
    // Open the repository file for the specified repository name
    ifstream repositoryFile(repoName + ".txt");
    if (!repositoryFile.is_open())
    {
        cout << "Error: Unable to open repository file for reading." << endl;
        Sleep(1000);
        return;
    }

    // Display contents of the repository file
    cout << "Contents of repository '" << repoName << "':" << endl;
    Sleep(1000);
    string projectName;
    int projectIndex = 0;

    while (getline(repositoryFile, projectName))
    {
        cout << ++projectIndex << ". " << projectName << endl;
    }
    repositoryFile.close();

    // Ask the user which project's contents they want to see
    int projectNumber;
    do
    {
        cout << "Enter the project number you want to display (0 to return to the main menu): ";
        cin >> projectNumber;
        if (projectNumber == 0)
        {
            return; // Return to main menu
        }
        else if (projectNumber < 1 || projectNumber > projectIndex)
        {
            cout << "Invalid project number. Please enter a valid project number." << endl;
            Sleep(1000);
        }
        else
        {
            // Open and display contents of the selected project file
            ifstream projectFile(repoName + ".txt");
            if (projectFile.is_open())
            {
                string projectLine;
                for (int i = 0; i < projectNumber; ++i)
                {
                    getline(projectFile, projectLine);
                }
                cout << "Contents of project '" << projectLine << "':" << endl;
                Sleep(1000);
                projectFile.close();
                ifstream projectContentFile(projectLine + ".txt");
                if (projectContentFile.is_open()) {
                    string line;
                    while (getline(projectContentFile, line))
                    {
                        cout << line << endl;
                    }
                    projectContentFile.close();
                    Sleep(1000);
                    system("pause");
                    return; // Exit function after displaying project contents
                }
                else
                {
                    cout << "Error: Unable to open project file for reading." << endl;
                    Sleep(1000);
                    return;
                }
            }
            else
            {
                cout<< "Error: Unable to open repository file for reading." << endl;
                Sleep(1000);
                return;
            }
        }
    } while (true); // Repeat until a valid project number is entered or the user chooses to return to the main menu
}



void Repository::setRepositoryName(const string& name)
{
    repositoryName = name;
}

string Repository::getRepositoryName(int index)
{
    if (index < 0 || index >= 100 || repositories[index] == nullptr) {
        cout << "Invalid repository index." << endl;
        Sleep(1000);
        return "";
    }
    // Return the repository name directly from the Repository class
    return repositoryName;
}

Friends::Friends()
{
    adjList = new FriendNode * [100];
    for (int i = 0; i < 100; ++i)
    {
        adjList[i] = nullptr;
    }
}

void Friends::addFriend(string userName, string friendName)
{
    // Get the hash value for the userName
    int hashVal = hashTable.gethash(userName);

    // Open the file to check if the friend name already exists
    string filename = "Friends" + to_string(hashVal) + ".txt";
    ifstream friendFile(filename);
    if (friendFile.is_open())
    {
        string name;
        while (getline(friendFile, name))
        {
            if (name == friendName)
            {
                cout << "Error: " << friendName << " is already in your friend list." << endl;
                Sleep(1000);

                friendFile.close();
                system("cls");
                return;
            }
        }
        friendFile.close();
    }

    // Create a new FriendNode with the given friendName
    FriendNode* newNode = new FriendNode(friendName);

    // Check if the adjacency list at hashVal index is empty
    if (adjList[hashVal] == nullptr)
    {
        adjList[hashVal] = newNode;
    }
    else
    {
        // If not empty, append the new FriendNode to the existing list
        newNode->nextPtr = adjList[hashVal];
        adjList[hashVal] = newNode;
    }

    // Store the friend name in a file named Friends+hashvalue.txt
    ofstream outFile(filename, ios::app);
    if (!outFile.is_open())
    {
        cout << "Error: Unable to open " << filename << " for writing." << endl;
        system("cls");
        return;
    }
    outFile << friendName << endl;
    outFile.close();
    cout << "Friend added successfully." << endl;
    Sleep(1000);

}

void Friends::unfollowFriend(string userName, string friendName)
{
    // Get the hash value for the userName
    int hashVal = hashTable.gethash(userName);

    // Open the file to read the friend names
    string filename = "Friends" + to_string(hashVal) + ".txt";
    ifstream friendFile(filename);
    if (!friendFile.is_open())
    {
        cout << "Error: Unable to open " << filename << " for reading." << endl;
        system("cls");
        return;
    }

    // Create a temporary file to store the updated content
    ofstream tempFile("temp.txt");
    if (!tempFile.is_open())
    {
        cout << "Error: Unable to create temporary file." << endl;
        friendFile.close();
        system("cls");
        return;
    }

    string name;
    bool found = false; // Flag to indicate if the friend to unfollow is found
    while (getline(friendFile, name))
    {
        if (name == friendName)
        {
            found = true; // Set the flag to true if the friend is found
        }
        else
        {
            tempFile << name << endl; // Write the friend name to the temporary file
        }
    }
    friendFile.close();
    tempFile.close();

    // If the friend to unfollow is not found, display a message and return
    if (!found)
    {
        cout << "Error: " << friendName << " is not in your friend list." << endl;
        Sleep(1000);
        system("cls");
        if (remove("temp.txt") != 0)
        {
            cout << "Error: Unable to delete temporary file." << endl;
            system("cls");
        }
        return;
    }

    // Replace the original file with the temporary file
    if (remove(filename.c_str()) != 0 || rename("temp.txt", filename.c_str()) != 0)
    {
        cout << "Error: Unable to update file." << endl;
        system("cls");
        return;
    }

    // Remove the friend node from the adjacency list
    FriendNode* current = adjList[hashVal];
    FriendNode* prev = nullptr;
    while (current != nullptr)
    {
        if (current->FriendName == friendName)
        {
            if (prev == nullptr)
            {
                adjList[hashVal] = current->nextPtr;
            }
            else
            {
                prev->nextPtr = current->nextPtr;
            }
            delete current;
            cout << "Friend " << friendName << " unfollowed successfully." << endl;

            Sleep(2000);
            system("pause");
            return;
        }
        prev = current;
        current = current->nextPtr;
    }
    if (!found)
        // If the friend node is not found in the adjacency list, display an error message
        cout << "Error: " << friendName << " is not in your friend list." << endl;
    system("cls");

}

void Friends::displayFriends(string userName)
{
    // Get the hash value for the userName
    int hashVal = hashTable.gethash(userName);

    cout << "Displaying Friends for " << userName << ":" << endl;
    cout << "------------------------------------------" << endl;
    Sleep(1000);
    // Open the file named Friends+hashvalue.txt
    string filename = "Friends" + to_string(hashVal) + ".txt";
    ifstream friendFile(filename);
    if (!friendFile.is_open())
    {
        cout << "Error: Unable to open " << filename << " for reading." << endl;
        return;
    }

    // Display friend names from the file
    string friendName;
    bool noFriends = true; // Flag to check if any friends are found
    while (getline(friendFile, friendName))
    {
        cout << "  - " << friendName << endl;
        noFriends = false; // Set the flag to false as at least one friend is found
        Sleep(1000); // Optional: Add delay between displaying friends
    }
    friendFile.close();

    // If no friends were found in the file, print a message
    if (noFriends) {
        cout << "No friends added yet." << endl;
    }

    cout << "------------------------------------------" << endl;
    system("pause");
}




Friends::~Friends()
{
    // Free memory allocated for each linked list in the adjacency list
    for (int i = 0; i < 100; ++i)
    {
        FriendNode* current = adjList[i];
        while (current != nullptr)
        {
            FriendNode* temp = current;
            current = current->nextPtr;
            delete temp;
        }
    }
    // Free memory allocated for the array of pointers
    delete[] adjList;
}

void printBoxMain()
{
    system("cls");
    cout << "\n\n\n";
    cout << "\t\t\t\t\t" << "================================" << endl;
    cout << "\t\t\t\t\t" << "|     Repository Management    |" << endl;
    cout << "\t\t\t\t\t" << "================================" << endl;
    cout << "\t\t\t\t\t" << "| 1. Register                  |" << endl;
    cout << "\t\t\t\t\t" << "| 2. Login                     |" << endl;
    cout << "\t\t\t\t\t" << "| 3. Exit                      |" << endl;
    cout << "\t\t\t\t\t" << "================================" << endl;
    cout << endl;
}

void printBox()
{
    system("cls");
    cout << "\n\n\n";
    cout << "\t\t\t\t\t" << "=====================================" << endl;
    cout << "\t\t\t\t\t" << "|      Repository Management        |" << endl;
    cout << "\t\t\t\t\t" << "=====================================" << endl;
    cout << "\t\t\t\t\t" << "| Menu:                             |" << endl;
    cout << "\t\t\t\t\t" << "| 1. Create new repository          |" << endl;
    cout << "\t\t\t\t\t" << "| 2. Add file to repository         |" << endl;
    cout << "\t\t\t\t\t" << "| 3. Delete file from repository    |" << endl;
    cout << "\t\t\t\t\t" << "| 4. Display contents of repository |" << endl;
    cout << "\t\t\t\t\t" << "| 5. Add Friend                     |" << endl;
    cout << "\t\t\t\t\t" << "| 6. Un_Follow Friend               |" << endl;
    cout << "\t\t\t\t\t" << "| 7. Display Friends                |" << endl;
    cout << "\t\t\t\t\t" << "| 8. Logout                         |" << endl;
    cout << "\t\t\t\t\t" << "=====================================" << endl;
    cout << endl;
}