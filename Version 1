import { useEffect, useState } from "react";

const STORAGE_KEY = "wer-was-bis-wann-vorlage";

const standardAufgaben = [
  { wer: "Ich", fach: "DE", was: "Aufsatz schreiben", bis: "Mittwoch", erledigt: false },
  { wer: "Ich", fach: "MA", was: "Seite 24 lösen", bis: "Donnerstag", erledigt: true },
  { wer: "Ich", fach: "RZG", was: "Arbeitsheft S. 125", bis: "Mittwoch", erledigt: false },
  { wer: "Ich", fach: "NT", was: "Experiment lernen", bis: "Freitag", erledigt: false },
];

function sicherLaden() {
  try {
    const gespeichert = localStorage.getItem(STORAGE_KEY);
    if (!gespeichert) return null;
    return JSON.parse(gespeichert);
  } catch {
    return null;
  }
}

export default function SchulVorlage() {
  const subjects = {
    DE: "bg-red-200 border-red-400",
    MA: "bg-blue-200 border-blue-400",
    E: "bg-yellow-200 border-yellow-400",
    F: "bg-green-200 border-green-400",
    RZG: "bg-orange-200 border-orange-400",
    NT: "bg-cyan-200 border-cyan-400",
    MU: "bg-purple-200 border-purple-400",
    BS: "bg-amber-200 border-amber-400",
    MI: "bg-gray-200 border-gray-400",
    WAH: "bg-orange-400 border-orange-600",
  };

  const fachListe = Object.keys(subjects);
  const gespeicherteDaten = sicherLaden();

  const [tasks, setTasks] = useState(gespeicherteDaten?.tasks || standardAufgaben);
  const [newTask, setNewTask] = useState({ wer: "Ich", fach: "DE", was: "", bis: "" });
  const [elternGesehen, setElternGesehen] = useState(Boolean(gespeicherteDaten?.elternGesehen));
  const [emailGesendet, setEmailGesendet] = useState(false);
  const [emailAdresse, setEmailAdresse] = useState(gespeicherteDaten?.emailAdresse || "");
  const [kopierHinweis, setKopierHinweis] = useState("");
  const [teilenLink, setTeilenLink] = useState("");

  useEffect(() => {
    const daten = { tasks, elternGesehen, emailAdresse };
    localStorage.setItem(STORAGE_KEY, JSON.stringify(daten));
  }, [tasks, elternGesehen, emailAdresse]);

  useEffect(() => {
    try {
      if (!window.location.hash.startsWith("#daten=")) return;
      const rohdaten = decodeURIComponent(window.location.hash.replace("#daten=", ""));
      const daten = JSON.parse(rohdaten);

      if (Array.isArray(daten.tasks)) setTasks(daten.tasks);
      setElternGesehen(Boolean(daten.elternGesehen));
      setEmailAdresse(daten.emailAdresse || "");
      setKopierHinweis("Geteilte Daten wurden geladen.");
    } catch {
      setKopierHinweis("Der Teilen-Link konnte nicht geladen werden.");
    }
  }, []);

  function aktuelleDaten() {
    return { tasks, elternGesehen, emailAdresse };
  }

  function formatiereSchweizerDatum(datum) {
    if (!datum) return "ohne Datum";

    try {
      return new Intl.DateTimeFormat("de-CH", {
        weekday: "long",
        day: "2-digit",
        month: "2-digit",
        year: "numeric",
      }).format(new Date(datum));
    } catch {
      return datum;
    }
  }

  function formatiereAufgabenText(aufgaben) {
    if (!aufgaben.length) return "Keine Aufgaben eingetragen.";

    return aufgaben
      .map((task) => {
        const fach = task.fach || "Ohne Fach";
        const was = task.was || "Ohne Aufgabe";
        const bis = formatiereSchweizerDatum(task.bis);
        const status = task.erledigt ? "Erledigt" : "Offen";
        return `${fach}: ${was} (${bis}) - ${status}`;
      })
      .join("\n");
  }

  function kompletterMailText() {
    return "Die Eltern haben die Aufgaben angesehen.\n\nAufgaben:\n" + formatiereAufgabenText(tasks);
  }

  function addTask() {
    if (!newTask.was.trim()) return;
    setTasks([...tasks, { ...newTask, erledigt: false }]);
    setNewTask({ wer: "Ich", fach: "DE", was: "", bis: "" });
  }

  function toggleTask(index) {
    const updated = [...tasks];
    updated[index].erledigt = !updated[index].erledigt;
    setTasks(updated);
  }

  function deleteTask(index) {
    if (elternGesehen) return;
    setTasks(tasks.filter((_, i) => i !== index));
  }

  function allesZuruecksetzen() {
    localStorage.removeItem(STORAGE_KEY);
    setTasks(standardAufgaben);
    setElternGesehen(false);
    setEmailAdresse("");
    setEmailGesendet(false);
    setKopierHinweis("");
    setTeilenLink("");
    window.location.hash = "";
  }

  function kopiereText(text, erfolgText) {
    setKopierHinweis(
      "Automatisches Kopieren ist in dieser Vorschau blockiert. Bitte den Text im Feld markieren und manuell kopieren."
    );
  }

  function kopiereMailText() {
    kopiereText(kompletterMailText(), "Mailtext wurde kopiert.");
  }

  function sendeEmail() {
    const betreff = encodeURIComponent("Hausaufgaben bestätigt");
    const text = encodeURIComponent(kompletterMailText());
    const empfaenger = encodeURIComponent(emailAdresse.trim());
    window.location.href = `mailto:${empfaenger}?subject=${betreff}&body=${text}`;
    setEmailGesendet(true);
  }

  function teilenLinkErstellen() {
    const daten = encodeURIComponent(JSON.stringify(aktuelleDaten()));
    const link = `${window.location.origin}${window.location.pathname}#daten=${daten}`;
    setTeilenLink(link);
    setKopierHinweis("Teilen-Link wurde erstellt. Bitte im Feld markieren und manuell kopieren.");
  }

  function datenHerunterladen() {
    const inhalt = JSON.stringify(aktuelleDaten(), null, 2);
    const blob = new Blob([inhalt], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "wer-was-bis-wann-daten.json";
    a.click();
    URL.revokeObjectURL(url);
  }

  function datenImportieren(event) {
    const datei = event.target.files?.[0];
    if (!datei) return;

    const reader = new FileReader();
    reader.onload = () => {
      try {
        const daten = JSON.parse(String(reader.result));
        if (!Array.isArray(daten.tasks)) throw new Error("Ungültige Datei");
        setTasks(daten.tasks);
        setElternGesehen(Boolean(daten.elternGesehen));
        setEmailAdresse(daten.emailAdresse || "");
        setKopierHinweis("Daten wurden importiert.");
      } catch {
        setKopierHinweis("Diese Datei konnte nicht importiert werden.");
      }
    };
    reader.readAsText(datei);
  }

  return (
    <div className="min-h-screen bg-gray-100 p-6">
      <div className="max-w-6xl mx-auto bg-white rounded-3xl shadow-xl p-6">
        <div className="flex flex-col md:flex-row md:items-center md:justify-between gap-3 mb-6">
          <h1 className="text-3xl font-bold">Wer – Was – Bis wann</h1>
          <button onClick={allesZuruecksetzen} className="bg-white border rounded-2xl px-4 py-2 text-sm font-semibold">
            Vorlage zurücksetzen
          </button>
        </div>

        <div className="bg-green-50 border border-green-200 rounded-2xl p-4 mb-6 text-green-800">
          <p className="font-semibold">Online-App-Version</p>
          <p className="text-sm mt-1">Einträge werden im Browser gespeichert. Mit Export, Import oder Teilen-Link kannst du die Daten weitergeben.</p>
        </div>

        <div className="bg-gray-50 rounded-3xl p-5 mb-8">
          <h2 className="text-xl font-bold mb-4">Neue Aufgabe eintragen</h2>

          <div className="grid grid-cols-1 md:grid-cols-5 gap-3">
            <input className="border rounded-2xl p-3" placeholder="Wer?" value={newTask.wer} onChange={(e) => setNewTask({ ...newTask, wer: e.target.value })} />

            <select className="border rounded-2xl p-3" value={newTask.fach} onChange={(e) => setNewTask({ ...newTask, fach: e.target.value })}>
              {fachListe.map((fach) => <option key={fach}>{fach}</option>)}
            </select>

            <input className="border rounded-2xl p-3 md:col-span-2" placeholder="Was ist zu tun?" value={newTask.was} onChange={(e) => setNewTask({ ...newTask, was: e.target.value })} />
            <input type="date" className="border rounded-2xl p-3" value={newTask.bis} onChange={(e) => setNewTask({ ...newTask, bis: e.target.value })} />
          </div>

          <button onClick={addTask} className="mt-4 bg-gray-900 text-white rounded-2xl px-5 py-3 font-semibold">
            + Aufgabe hinzufügen
          </button>
        </div>

        <div className="overflow-x-auto">
          <table className="w-full border-collapse">
            <thead>
              <tr className="bg-gray-200 text-left">
                <th className="p-3 rounded-l-2xl">Erledigt</th>
                <th className="p-3">Wer</th>
                <th className="p-3">Fach</th>
                <th className="p-3">Was</th>
                <th className="p-3">Bis wann</th>
                <th className="p-3 rounded-r-2xl">Löschen</th>
              </tr>
            </thead>
            <tbody>
              {tasks.map((task, index) => (
                <tr key={index} className={`border-b ${subjects[task.fach] || "bg-white"} ${task.erledigt ? "opacity-60 line-through" : ""}`}>
                  <td className="p-3 text-center"><input type="checkbox" checked={task.erledigt} onChange={() => toggleTask(index)} className="w-6 h-6" /></td>
                  <td className="p-3">{task.wer}</td>
                  <td className="p-3 font-semibold">{task.fach}</td>
                  <td className="p-3">{task.was}</td>
                  <td className="p-3">{formatiereSchweizerDatum(task.bis)}</td>
                  <td className="p-3">
                    <button onClick={() => deleteTask(index)} disabled={elternGesehen} className={`border rounded-xl px-3 py-1 ${elternGesehen ? "bg-gray-200 text-gray-400 cursor-not-allowed" : "bg-white"}`}>
                      {elternGesehen ? "Gesperrt" : "Löschen"}
                    </button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>

          <div className="mt-6 bg-blue-50 border-2 border-blue-200 rounded-2xl p-4">
            <h3 className="font-bold text-lg mb-3">Bestätigung der Eltern</h3>

            <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
              <div>
                <p className="text-sm text-gray-600 mb-1">Datum</p>
                <input className="w-full bg-white border rounded-xl p-3 min-h-[48px]" placeholder="Hier schreiben" />
              </div>
              <div>
                <p className="text-sm text-gray-600 mb-1">Unterschrift Eltern</p>
                <input className="w-full bg-white border rounded-xl p-3 min-h-[48px]" placeholder="Hier schreiben" />
              </div>
              <div>
                <p className="text-sm text-gray-600 mb-1">Gesehen ☑</p>
                <div className="flex items-center justify-center bg-white border rounded-xl p-3 min-h-[48px] text-2xl">
                  <input type="checkbox" checked={elternGesehen} onChange={(e) => setElternGesehen(e.target.checked)} className="w-6 h-6" />
                </div>
              </div>
            </div>

            <p className="text-sm text-gray-500 mt-3">Sobald „Gesehen“ angehakt ist, können Aufgaben nicht mehr gelöscht werden.</p>

            {elternGesehen && (
              <div className="mt-4 bg-white border rounded-2xl p-4">
                <p className="font-semibold mb-2">Mailadresse eintragen</p>
                <input className="w-full border rounded-xl p-3 mb-3" placeholder="z.B. lehrperson@schule.ch" value={emailAdresse} onChange={(e) => setEmailAdresse(e.target.value)} />

                <div className="flex flex-col md:flex-row gap-3">
                  <button onClick={sendeEmail} className="bg-blue-600 text-white rounded-2xl px-5 py-3 font-semibold">📧 Mailprogramm öffnen</button>
                  <button onClick={kopiereMailText} className="bg-gray-900 text-white rounded-2xl px-5 py-3 font-semibold">📋 Mailtext kopieren</button>
                </div>

                <div className="mt-4">
                  <p className="font-semibold mb-2">Mailtext zum Kopieren</p>
                  <textarea readOnly className="w-full border rounded-xl p-3 min-h-[160px]" value={kompletterMailText()} onFocus={(e) => e.target.select()} />
                </div>
              </div>
            )}

            {emailGesendet && <p className="text-green-600 mt-3 font-semibold">Mail wurde vorbereitet. Bitte im Mailprogramm noch auf „Senden“ klicken.</p>}
          </div>
        </div>

        <div className="mt-8 bg-indigo-50 border border-indigo-200 rounded-3xl p-5">
          <h2 className="text-xl font-bold mb-3">Teilen und Speichern</h2>
          <div className="flex flex-col md:flex-row gap-3 mb-4">
            <button onClick={teilenLinkErstellen} className="bg-indigo-600 text-white rounded-2xl px-5 py-3 font-semibold">🔗 Teilen-Link anzeigen</button>
            <button onClick={datenHerunterladen} className="bg-white border rounded-2xl px-5 py-3 font-semibold">⬇️ Daten herunterladen</button>
            <label className="bg-white border rounded-2xl px-5 py-3 font-semibold cursor-pointer text-center">
              ⬆️ Daten importieren
              <input type="file" accept="application/json" onChange={datenImportieren} className="hidden" />
            </label>
          </div>

          {teilenLink && (
            <textarea readOnly className="w-full border rounded-xl p-3 min-h-[80px]" value={teilenLink} onClick={(e) => e.target.select()} onFocus={(e) => e.target.select()} />
          )}

          {teilenLink && (
            <p className="text-sm text-gray-600 mt-2">
              Klicke in das Feld, markiere den Link und kopiere ihn mit Ctrl+C oder Rechtsklick → Kopieren.
            </p>
          )}

          {kopierHinweis && <p className="text-sm text-orange-700 mt-3 font-semibold">{kopierHinweis}</p>}
        </div>

        <div className="mt-8 grid grid-cols-2 md:grid-cols-10 gap-3">
          <div className="bg-red-200 rounded-2xl p-3 text-center font-semibold">DE</div>
          <div className="bg-blue-200 rounded-2xl p-3 text-center font-semibold">MA</div>
          <div className="bg-yellow-200 rounded-2xl p-3 text-center font-semibold">E</div>
          <div className="bg-green-200 rounded-2xl p-3 text-center font-semibold">F</div>
          <div className="bg-orange-200 rounded-2xl p-3 text-center font-semibold">RZG</div>
          <div className="bg-cyan-200 rounded-2xl p-3 text-center font-semibold">NT</div>
          <div className="bg-purple-200 rounded-2xl p-3 text-center font-semibold">MU</div>
          <div className="bg-amber-200 rounded-2xl p-3 text-center font-semibold">BS</div>
          <div className="bg-gray-200 rounded-2xl p-3 text-center font-semibold">MI</div>
          <div className="bg-orange-400 rounded-2xl p-3 text-center font-semibold text-white">WAH</div>
        </div>

        <div className="mt-8 bg-gray-50 rounded-2xl p-5">
          <h2 className="text-xl font-bold mb-3">So funktioniert es</h2>
          <ul className="space-y-2 text-gray-700">
            <li>1. Aufgaben eintragen und abhaken</li>
            <li>2. Eltern haken „Gesehen“ ab</li>
            <li>3. Mailprogramm öffnen oder Mailtext kopieren</li>
            <li>4. Mit „Teilen-Link erstellen“ kannst du den aktuellen Stand weitergeben</li>
            <li>5. Mit „Daten herunterladen“ kannst du eine Sicherung speichern</li>
            <li>6. Mit „Daten importieren“ kann jemand diese Sicherung wieder öffnen</li>
          </ul>
        </div>
      </div>
    </div>
  );
}
