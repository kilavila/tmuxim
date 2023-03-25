```bash
# README:
#  _____                     _           
# |_   _| __ ___  _   ___  _(_)_ __ ___  
#   | || '_ ` _ \| | | \ \/ / | '_ ` _ \ 
#   | || | | | | | |_| |>  <| | | | | | |
#   |_||_| |_| |_|\__,_/_/\_\_|_| |_| |_|
# ------------------------------------------
# Author: Christer Kilavik
# https://github.com/kilavila/tmuxim
# ------------------------------------------
# This script requires the following packages to be installed:
# tmux - https://github.com/tmux/tmux/wiki/Installing
# fzf  - https://github.com/junegunn/fzf
# tr   - (should be installed by default as part of coreutils)

# USAGE:
# $ tmuxim
# Add the following line in your ~/.bashrc to get Ctrl+t to run tmuxim:
# bind '"\C-t":"tmuxim\n"'
# Add the following lines to your ~/.bashrc to get a better fzf experience:
# export FZF_DEFAULT_COMMAND='rg --files --no-ignore --hidden --follow --glob "!.git/*"'
# export FZF_DEFAULT_OPTS="--height 40% --margin 0.5% -1 --layout=reverse --multi --cycle --prompt='Search > ' --info=inline --border=top --color=border:bright-yellow --border-label='╢ FZF ╟' --border-label-pos=-3 --color=label:italic:bright-yellow --preview-label='╢ Preview ╟' --preview-label-pos=3 --preview='[[ \$(file --mime {}) =~ binary ]] && echo {} is a binary file || (bat --style=numbers --color=always {} || cat {}) 2> /dev/null | head -300'"


# INFO:
# Prints current path and gets current directory name only
# and uses this as the name of the session

tmuxim_autoname_session(){
  NAME=$(pwd | sed -E 's/^.*\/(.*)$/\1/g')
  tmux new -s "$NAME"

  return
}

# INFO:
# Uses read to get user input and uses this as the name of the session
tmuxim_manual_name_session(){
  read -rp "Enter session name: " NAME

  if [ -z "$NAME" ]; then
    return
  else
    tmux new -s "$NAME"
  fi

  return
}

# INFO:
# Selection menu to automatically name a session
# or to manually name a session
tmuxim_new_session(){
  MENU="Autoname session,Manual name session"
  AUTO_NAME_SESSION=$(echo "$MENU" | tr ',' '\n' | fzf --prompt="Select > " --border-label='╢ Tmux new session ╟' --preview-window=hidden)

  if [ -z "$AUTO_NAME_SESSION" ]; then
    return
  elif [[ "$AUTO_NAME_SESSION" == "Autoname session" ]]; then
    tmuxim_autoname_session
  elif [[ "$AUTO_NAME_SESSION" == "Manual name session" ]]; then
    tmuxim_manual_name_session
  fi

  return
}

# INFO:
# Selection menu to attach to a session
# if only 1 session is running it will attach to it automatically
tmuxim_attach_session(){
  SESSION=$(tmux ls | fzf --prompt="Select > " --border-label='╢ Tmux attach session ╟' --preview-window=hidden)

  if [ -z "$SESSION" ]; then
    return
  else
    NAME=$(echo "$SESSION" | sed -E 's/^([^:]).*$/\1/g')
    tmux a -t "$NAME"
  fi

  return
}

# INFO:
# Selection menu to kill a session
# if only 1 session is running it will kill it automatically
# if multiple sessions are running it will send the user back to main menu
# if all sessions are killed then send user to new session menu
tmuxim_kill_session(){
  SESSION=$(tmux ls | fzf --prompt="Select > " --border-label='╢ Tmux kill session ╟' --preview-window=hidden)

  if [ -z "$SESSION" ]; then
    return
  else
    NAME=$(echo "$SESSION" | sed -E 's/^([^:]).*$/\1/g')
    tmux kill-session -t "$NAME"
  fi

  SESSIONS=$(tmux ls | wc -l)

  if [ "$SESSIONS" -eq 0 ]; then
    tmuxim_new_session
  else
    tmuxim
  fi
}

MAIN_MENU="New Session,Attach session,Kill session"

# INFO:
# Main selection menu for creaing a new session, attach to a session or to kill a session
# if no sessions are running send user to new session menu
tmuxim(){
  SESSIONS=$(tmux ls | wc -l)

  if [ "$SESSIONS" -eq 0 ]; then
    tmuxim_new_session
    return
  fi

  tmux ls

  SELECTION=$(echo "$MAIN_MENU" | tr ',' '\n' | fzf --prompt="Select > " --border-label='╢ Tmux ╟' --preview-window=hidden)

  if [ -z "$SELECTION" ]; then
    return
  elif [[ "$SELECTION" == "New Session" ]]; then
    tmuxim_new_session
  elif [[ "$SELECTION" == "Attach session" ]]; then
    tmuxim_attach_session
  elif [[ "$SELECTION" == "Kill session" ]]; then
    tmuxim_kill_session
  fi

  return
}
```
