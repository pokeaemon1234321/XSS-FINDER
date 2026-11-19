Here’s the full walkthrough, from creating the file to running it:

Step 1 — Create the script file

bash
nano xss_check.sh

(or use vim xss_check.sh, or create it in a text editor/VS Code if you prefer — just make sure it ends up in your current working directory, e.g. ~/waybackurlsbash/)

Step 2 — Paste the script content

Paste this entire block in:

bash
#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: $0 <urls_file>"
    exit 1
fi

> success.txt   # clear/create output file fresh each run

grep '=' "$1" | sed 's/=.*/=rohit<>/g' | sort -u | while read -r host; do
    curl -sk --connect-timeout 5 --max-time 10 \
        -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
        "$host" -o /tmp/out.html
    code=$?

    if [ "$code" -ne 0 ]; then
        echo -e "$host -> \e[33mFAILED (curl exit $code)\e[0m"
    elif grep -q 'rohit<>' /tmp/out.html; then
        echo -e "$host -> \e[32mVULNERABLE\e[0m"
        echo "$host" >> success.txt
    else
        echo -e "$host -> \e[31mNOT VULNERABLE\e[0m"
    fi

    sleep 1
done

echo ""
echo "Done. Vulnerable URLs saved to success.txt"

Step 3 — Save and exit

If using nano: press Ctrl + O, then Enter to save, then Ctrl + X to exit.
If using vim: press Esc, type :wq, press Enter.

Step 4 — Make the script executable

bash
chmod +x xss_check.sh

Step 5 — Confirm your input file exists

Make sure your URL list (e.g. test.txt) is in the same directory, or note its full path:

bash
ls -l test.txt

Step 6 — Run the script

bash
./xss_check.sh test.txt

(Replace test.txt with whatever your actual URL list filename is.)

Step 7 — Watch the output

You’ll see each URL print live as:

🟡 FAILED — curl couldn’t connect/timed out
🟢 VULNERABLE — payload reflected in response
🔴 NOT VULNERABLE — payload not found

Step 8 — Check saved results after it finishes

bash
cat success.txt

This will list only the URLs flagged as vulnerable.

If you get a “permission denied” or “bad interpreter” error:

Permission denied → re-run step 4 (chmod +x).
Bad interpreter / \r error → your file might have Windows line endings. Fix with:
bash
sed -i 's/\r$//' xss_check.sh
