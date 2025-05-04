Jellyfin Toast (admin to client Notification)

okay so here is a mod to put a notification in the top right and keep it there until they click the x in the toast or have it time out 


![Screenshot 2025-05-04 183512](https://github.com/user-attachments/assets/97b27d75-7713-4bd4-87e4-303e732b488b)

![Screenshot 2025-05-04 183519](https://github.com/user-attachments/assets/1c89a0d4-9f86-490d-ada6-d01c603792af)


add the following in your `index.html` but inside the `body` tags 

````


<script>
document.addEventListener('DOMContentLoaded', async function() {
    const container = document.createElement('div');
    container.id = 'notification-container';
    container.style.position = 'fixed';
    container.style.top = '20px';
    container.style.right = '15px';
    container.style.zIndex = '9999';
    container.style.display = 'flex';
    container.style.flexDirection = 'column';
    container.style.alignItems = 'flex-end';
    container.style.pointerEvents = 'none';
    document.body.appendChild(container);

    const style = document.createElement('style');
    style.textContent = `
    @keyframes fadeIn {
        from { opacity: 0; transform: translateY(-10px); }
        to { opacity: 1; transform: translateY(0); }
    }
    .notification {
        background: #111;
        color: #ddd;
        padding: 15px;
        margin-bottom: 10px;
        border-radius: 18px;
        max-width: 350px;
        box-shadow: 0 2px 8px rgba(0,0,0,0.5);
        word-break: break-word;
        font-family: Arial, sans-serif;
        position: relative;
        animation: fadeIn 0.5s ease forwards;
        opacity: 0;
        pointer-events: auto;
    }
    .notification .close-btn {
        position: absolute;
        top: -7px;
        right: 1px;
        cursor: pointer;
        font-size: 30px;
        color: #bbb;
    }
    `;
    document.head.appendChild(style);

    function createNotification(message, id) {
        const notif = document.createElement('div');
        notif.className = 'notification';

        const closeBtn = document.createElement('span');
        closeBtn.className = 'close-btn';
        closeBtn.innerHTML = '&times;';

        closeBtn.onclick = () => {
            notif.remove();
            localStorage.setItem('notification_read_' + id, 'true');
        };

        notif.appendChild(closeBtn);

        const msg = document.createElement('div');
        msg.textContent = message;
        notif.appendChild(msg);

        container.appendChild(notif);

        // Auto-hide after 50 seconds
        setTimeout(() => {
            if (notif.parentNode) {
                notif.remove();
                localStorage.setItem('notification_read_' + id, 'true');
            }
        }, 50000);
    }

    try {
        const response = await fetch('/web/notification.txt', { cache: 'no-store' });
        if (!response.ok) return;

        const text = await response.text();
        const lines = text.split('\n').map(line => line.trim()).filter(line => line.length > 0);

        lines.forEach((line, index) => {
            const parts = line.split('|');
            if (parts.length < 2) return;
            const id = parts[0].trim();
            const message = parts.slice(1).join('|').trim();

            if (!localStorage.getItem('notification_read_' + id)) {
                setTimeout(() => createNotification(message, id), index * 300);
            }
        });
    } catch (err) {
        console.error('Notification fetch failed:', err);
    }
});
</script>
````

now create a notification.txt inside your web root (thats next to your index.html file) it should contain one notification per line you can have as many as you like and leave ones in there for aslong as you want

example 
````
UNIQUEIDHERE|this notification is a unique notification that each web client will only see once and doesnt spam them every login
````

see how there is a UNIQUEIDHERE this needs to be something unique per line so it saves that per client so it doesnt keep spamming the user the same notification about maintenance or whatever on each page reload

reason for this mod is simple, i had a banner.. but i got annoyed and wanted something more.... sleek and something that doesnt instantly annoy some users.
