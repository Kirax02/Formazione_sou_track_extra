# Esercizio Ansible: Jinja Templates
## Obiettivi
1. Creare un playbook Ansible che aggiunga in append sul file /etc/ security/limits.conf alcuni settings per un’utente. In ambiente di produzione dobbiamo imporre un numero massimo di file aperti pari a 10000, mentre in ambiente di collaudo e sviluppo 1000.  

2. Supponiamo che in /etc/security/access.conf ci sia un’ultima riga che impedisce l’accesso agli utenti non esplicitamente autorizzati (“- : ALL : ALL”). Creare un playbook Ansible che aggiunga una lista di utenti in whitelist anteponendosi a tale riga (hint: utilizzare l’opzione insertbefore del modulo blockinfile).  

3. [OPZIONALE] Trovare un possibile utilizzo dei Jinja templates per la parametrizzazione di almeno uno dei seguenti oggetti:  
• Dockerfile  
• File yaml rappresentante un pod o un deployment Kubernetes   
• Virtual host di apache

## Soluzione
### Obiettivo numero 1
Per completare questo obiettivo, ho creato una directory per i template Jinja, all'interno della quale si trova il file `limits.j2`. Questo template contiene le impostazioni per gli ambienti di produzione, collaudo e sviluppo.  
Le seguenti task Ansible gestiscono il processo:
```yaml
  tasks:
    - name: Genera il file limits.conf con il template Jinja
      ansible.builtin.template:
        src: templates/limits.j2
        dest: /tmp/limits.conf

    - name: Aggiungi impostazioni a /etc/security/limits.conf
      ansible.builtin.blockinfile:
        path: /etc/security/limits.conf
        block: "{{ lookup('ansible.builtin.file', '/tmp/limits.conf') }}"
        insertafter: EOF
        create: yes
```
1. La prima task genera il file temporaneo `limits.conf` utilizzando il template Jinja. 
2. La seconda task aggiunge le impostazioni generate al file `/etc/security/limits.conf`.
### Obiettivo numero 2
Per il secondo obiettivo, ho implementato la seguente task:  
```yaml
    - name: Aggiungi whitelist a /etc/security/access.conf
      ansible.builtin.blockinfile:
        path: /etc/security/access.conf
        block: |
          {% for user in whitelist_users %}
          + : {{ user }} : ALL
          {% endfor %}
        insertbefore: "- : ALL : ALL"
        create: yes
```
Questa task utilizza il modulo blockinfile per:

1. Iterare sugli utenti definiti nella variabile whitelist.
2. Aggiungere una riga per ciascun utente in formato + : : ALL.
3. Inserire le righe generate prima della direttiva - : ALL : ALL, utilizzando l'opzione insertbefore.
### Obiettivo numero 3
Per il terzo obiettivo ho aggiunto la seguente task:
```yaml
    - name: Genera il Dockerfile dinamico
      ansible.builtin.template:
        src: templates/Dockerfile.j2
        dest: /home/vagrant/Dockerfile
```
Questa task genera un Dockerfile personalizzato utilizzando un template Jinja `Dockerfile.j2`. Ciò consente di parametrizzare facilmente le configurazioni.
