In the project you developed further, what did a fresh session get wrong or ask about that an earlier round had already settled? What was missing from the repository that would have prevented it?
    - I was reading through the notes Claude gives in the terminal while it works and saw that the round 5 session got the STATE strings wrong in its browser smoke test. Its test script compared states to 'shop', 'menu' and 'playing', so four checks failed. It then had to look up the real values  which were 'SHOP', 'MENU' and 'PLAYING'. Rounds 2 to 4 had already written working smoke tests against those same values. Nothing in the repo said STATE values were uppercase until the CLAUDE.md update after Round 5. A committed script such as tests/smoke/smoke.mjs would have prevented it.

In the round that changed an earlier requirement, what broke? Did the tests catch it, and does the decision record explain the change to someone who was not there?
    -Round 5 changed three things, the phase power-up went from 30s to 5s, skins now save, and prices for skins went up from 100/200 shards to 300/600 shards. 12 earlier tests failed at once, since they weren't updated to match the new requirements. Decision records explain what happened to someone who wasn't there for the most part 0007 quotes the requirement, names what it supersedes, and lists rejected options with reasons. 0002's status line points to it. But it's dense ("union merge", "Y-statement", test filenames), and it never says why the user reversed R10. It just restates the new requirement. For the phase change there was nothing written

Name one test that caught a real problem, or one test the assistant wrote that checked nothing useful. Quote it.
    - (tests/state-highscore.test.js:129):
test('unknown state or action → NONE', () => {
  for (const a of ALL_ACTIONS) assert.equal(commandFor('BOGUS', a, 99, CONFIG), CMD.NONE);
  In Phase 7, adding mute made commandFor return MUTE even for an unknown state. The session said: "An existing test caught a real edge case… Fixing the logic (the test is right)". This was an edge case but still good that it was caught

Choose one commit you would now write differently, and say how.
    -"Save bought and equipped characters, and raise prices to 300 / 600 (R19, R20)" covers two requirements in nine files. It also rewrites the Round 3 tests in the same commit. I'd split it into three:
1. Raise character prices to 300 / 600 (R20): the config change, plus making the R7 tests read prices from CONFIG.
2. Replace R10 tests with R19: characters are saved: the old tests swapped for new ones that fail until the next commit.
3. Save owned and equipped characters under neonWasteland.characters (R19): the implementation.

Choose one function in the project you developed further that you did not read while building it. Without an assistant open, read it and write what you think it does, in two or three sentences. Then ask an assistant, and say what you got right and wrong.
    - Function: mergeStoredShop(shop, storage) in js/logic/shop.js:88
    -My guess before reading: It loads the characters saved in localStorage and combines them with the shop in memory, so a skin bought in another tab shows up here. It probably also takes the equipped character from storage and saves the merged result.
    -What it actually does: It reads the saved list through loadShop, and for each character marks it owned in shop if storage says it's owned and shop doesn't. It returns true if it added anything. It changes shop in place and only ever adds, so a character can never be un-owned. It doesn't touch the equipped character and it never writes to storage. saveShop does the writing, and it calls mergeStoredShop first. If the saved data is missing or garbage, loadShop returns a Hacker-only shop, which adds nothing.
    -Right: that it combines the stored and in-memory owned lists so another tab's purchase isn't lost.
    -Wrong: that it changes the equipped character and that it saves. That split is the point of the design: whichever tab you click in keeps its own choice of runner, and storage is only written on a shop click.