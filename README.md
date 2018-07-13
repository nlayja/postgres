# postgres

SELECT users.id
FROM users
INNER JOIN messages 
ON messages.send_by = users.id OR messages.receive_by = users.id
GROUP BY users.id
ORDER BY users.id

SELECT users.id, users.last_name, users.first_name, MAX(messages.send_at) AS OK
FROM "users"  
INNER JOIN messages 
ON messages.send_by = users.id OR messages.receive_by = users.id
WHERE messages.send_by = 59 or messages.receive_by = 59
GROUP BY users.id
ORDER BY OK DESC





SELECT users.id, users.last_name, users.first_name, messages.send_at 
FROM users
INNER JOIN messages
ON messages.send_by = users.id
OR messages.receive_by = users.id
GROUP BY users.id, messages.send_at
ORDER BY messages.send_at DESC



SELECT DISTINCT users.id, users.* 
FROM "users" 
INNER JOIN messages 
ON (messages.send_by = users.id 
OR messages.receive_by = users.id) 
ORDER BY messages.send_at DESC

def with_messages
  # byebug
  @message = User.joins("INNER JOIN messages ON messages.send_by = users.id OR messages.receive_by = users.id").where("messages.send_by = ? OR messages.receive_by = ?", current_user.id, current_user.id).order('recent_day DESC').select('users.id, MAX (messages.send_at) AS recent_day, users.last_name, users.first_name, nickname, email, birthdate, phone, gender, alternate_phone, emergency_contact_name, emergency_contact_phone, usapa_member_num, canada_member_num, is_instructor, iptpa_certified_instructor, usapa_certified_ref, usapa_ambassador, play_singles, play_doubles, play_mixed, shirt_size, stripe_account_status, stripe_account_type, stripe_user_id, publishable_key, secret_key').group('users.id')
  json_response(@message)
end
