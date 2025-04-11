## Steps to Launch an EC2 Instance, Install Flask, and Run a Simple App

This guide outlines the steps to launch an Ubuntu 24.04 EC2 instance, configure its security group, connect via SSH, install necessary dependencies, create a Flask application, set up a virtual environment, install Flask and Gunicorn, and finally run the application.

**1. Launch an EC2 Instance:**

* **(Through AWS Console - Assuming you are logged into your AWS account):**
    * Navigate to the **EC2** service dashboard.
    * Click on **Instances** in the left-hand menu.
    * Click the **Launch instances** button.
    * **Choose an Amazon Machine Image (AMI):** Search for and select **Ubuntu 24.04 LTS**.
    * **Choose an Instance Type:** Select an appropriate instance type (e.g., `t2.micro` for testing).
    * **Configure Instance Details:** Accept the defaults or customize as needed.
    * **Add Storage:** Accept the default or adjust the size.
    * **Configure Security Group:**
        * You can either select an existing security group or create a new one.
        * **Crucially, ensure the following inbound rules are configured:**
            * **Type:** SSH, **Protocol:** TCP, **Port Range:** 22, **Source:** 0.0.0.0/0
            * **Type:** Custom TCP, **Protocol:** TCP, **Port Range:** 5001, **Source:** 0.0.0.0/0
        * Review and click **Create security group** if needed.
    * **Review and Launch:** Review your instance configuration and click **Launch**.
    * **Select or Create a Key Pair:**
        * Choose an existing key pair from the dropdown or select **Create a new key pair**.
        * Enter a name for your key pair (e.g., `python`).
        * Click **Create new key pair**.
        * **Download the private key file (`python.pem`) and store it in a secure location (e.g., your `Downloads` directory).** You will need this file to connect to your instance via SSH.
    * Click **Launch Instances**.

**2. Connect to the EC2 Instance via SSH:**

* Open your local terminal.
* Navigate to the directory where you saved your private key file (e.g., `Downloads`):
    ```bash
    cd Downloads
    ```
* Use the `ssh` command to connect to your EC2 instance. Replace `"python.pem"` with the actual name of your key pair file and `ubuntu@ec2-44-220-153-114.compute-1.amazonaws.com` with the public DNS or IP address of your EC2 instance:
    ```bash
    ssh -i "python.pem" ubuntu@ec2-44-220-153-114.compute-1.amazonaws.com
    ```
    * You might see a warning about the authenticity of the host. Type `yes` and press Enter to continue.

**3. Update and Install Dependencies:**

* Once connected to your EC2 instance, run the following commands to update the package lists and install necessary dependencies:
    ```bash
    sudo apt update
    sudo apt install -y libmariadb-dev gcc g++ build-essential python3-dev dh-python python3-venv
    ```

**4. Create Flask App:**

* Make a new directory for your Flask application:
    ```bash
    mkdir app
    cd app
    ```
* Create a file named `main.py` using a text editor like `vim`:
    ```bash
    vim main.py
    ```
* Paste the following Python code into `main.py` and save the file (in `vim`, press `i` to enter insert mode, paste the code, press `Esc`, type `:wq` and press Enter):

    ```python
    from flask import Flask, jsonify
    import random

    app = Flask(__name__)

    quotes = [
        {"id": 1, "author": "Nelson Mandela", "quote": "The greatest glory in living lies not in never falling, but in rising every time we fall."},
        {"id": 2, "author": "Walt Disney", "quote": "The way to get started is to quit talking and begin doing."},
        {"id": 3, "author": "Steve Jobs", "quote": "Your time is limited, so don't waste it living someone else's life."},
        {"id": 4, "author": "Oprah Winfrey", "quote": "If you look at what you have in life, you'll always have more. If you look at what you don't have in life, you'll never have enough."},
        {"id": 5, "author": "Eleanor Roosevelt", "quote": "The future belongs to those who believe in the beauty of their dreams."},
    ]

    @app.route('/quote', methods=['GET'])
    def get_random_quote():
        random_quote = random.choice(quotes)
        return jsonify(random_quote)

    if __name__ == "__main__":
        app.run(port=5001, debug=True)
    ```

**5. Setup Virtual Environment and Install Flask:**

* Create a virtual environment to isolate your project dependencies:
    ```bash
    python3 -m venv myenv
    ```
* Activate the virtual environment:
    ```bash
    source myenv/bin/activate
    ```
    * Your terminal prompt should now be prefixed with `(myenv)`.
* Install the Flask framework:
    ```bash
    pip install flask
    ```

**6. Create WSGI File:**

* Create a file named `wsgi.py`:
    ```bash
    vim wsgi.py
    ```
* Paste the following code into `wsgi.py` and save:

    ```python
    from main import app as application

    if __name__ == "__main__":
        application.run(host='0.0.0.0', port=5001)
    ```

**7. Install Gunicorn and Run the App:**

* Install the Gunicorn WSGI HTTP server:
    ```bash
    pip install gunicorn
    ```
* Run your Flask application using Gunicorn. This makes your application accessible from outside the instance:
    ```bash
    gunicorn --bind 0.0.0.0:5001 wsgi:application
    ```
    * The `--bind 0.0.0.0:5001` option tells Gunicorn to listen on all network interfaces (0.0.0.0) and port 5001.
    * `wsgi:application` tells Gunicorn to load the `application` object from the `wsgi.py` file.

**8. Verify the Flask App is Live:**

* Open a web browser and navigate to the public IP address or public DNS name of your EC2 instance, followed by port `5001`. For example: `http://ec2-44-220-153-114.compute-1.amazonaws.com:5001/quote`
* You should see a random motivational quote in JSON format:

    ```json
    {
        "author": "Walt Disney",
        "id": 2,
        "quote": "The way to get started is to quit talking and begin doing."
    }
    ```
    * The quote will be different each time you refresh the page.

Your Flask application is now successfully deployed and accessible on your EC2 instance. Remember to deactivate your virtual environment when you are finished working on the project:

```bash
deactivate