# 6. Useful Notes

[← Back to overview](../README.md) · Previous: [5. Creating an Exam](05-creating-an-exam.md)

- If an exam should have **no quit password**, remove it under *Configurations → Exam Configuration → Edit Exam Configuration*, not under *Exam Administration → Exam → Edit Exam*.
- When the SEB lock is active, a button to close SEB is shown automatically after the exam.
- Finished exams appear under *Monitoring → Finished Exams* about 1 hour after the Moodle quiz closes.
- To allow the clipboard on Windows clients, enable the clipboard in the templates (not only in the configs the templates were created from); the default clipboard value in the database may also need to be set to `0` (allow).
- The quit password shown in the Moodle quiz settings can be hidden by adjusting the plugin (see `rule.php` in the [plugin repository](https://github.com/ethz-let/moodle-quizaccess_sebserver/blob/MOODLE_401_STABLE/rule.php)).
